# Progressive Path Tracing with Explicit Light Sampling

https://computergraphics.stackexchange.com/questions/5152/progressive-path-tracing-with-explicit-light-sampling

I understood the logic behind the importance sampling for the BRDF part. 
However, when it comes to sampling light sources explicitly, all becomes confusing. 
For example, if I have one point light source in my scene and if I directly sample it at each frame constantly, should I count it as one more sample for the monte carlo integration? 
That is, I take one sample from cosine-weighted distribution and other from the point light. Is it two samples in total or just one? 
Also, should I divide the radiance coming from the direct sample to any term?

---
# a backwards path tracer
There are multiple areas in path tracing that can be importance sampled. 
In addition, each of those areas can also use Multiple Importance Sampling, first proposed in Veach and Guibas's 1995 paper. 
To better explain, let's look at a backwards path tracer:

```
void RenderPixel(uint x, uint y, UniformSampler *sampler) {
    Ray ray = m_scene->Camera->CalculateRayFromPixel(x, y, sampler);

    float3 color(0.0f);
    float3 throughput(1.0f);
    SurfaceInteraction interaction;

    // Bounce the ray around the scene
    const uint maxBounces = 15;
    for (uint bounces = 0; bounces < maxBounces; ++bounces) {
        m_scene->Intersect(ray);

        // The ray missed. Return the background color
        if (ray.GeomID == INVALID_GEOMETRY_ID) {
            color += throughput * m_scene->BackgroundColor;
            break;
        }

        // Fetch the material
        Material *material = m_scene->GetMaterial(ray.GeomID);

        // The object might be emissive. If so, it will have a corresponding light
        // Otherwise, GetLight will return nullptr
        Light *light = m_scene->GetLight(ray.GeomID);

        // If we hit a light, add the emission
        if (light != nullptr) {
            color += throughput * light->Le();
        }

        interaction.Position = ray.Origin + ray.Direction * ray.TFar;
        interaction.Normal = normalize(m_scene->InterpolateNormal(ray.GeomID, ray.PrimID, ray.U, ray.V));
        interaction.OutputDirection = normalize(-ray.Direction);

        // Get the new ray direction
        // Choose the direction based on the bsdf        
        material->bsdf->Sample(interaction, sampler);
        float pdf = material->bsdf->Pdf(interaction);

        // Accumulate the weight
        throughput = throughput * material->bsdf->Eval(interaction) / pdf;

        // Shoot a new ray

        // Set the origin at the intersection point
        ray.Origin = interaction.Position;

        // Reset the other ray properties
        ray.Direction = interaction.InputDirection;
        ray.TNear = 0.001f;
        ray.TFar = infinity;


        // Russian Roulette
        if (bounces > 3) {
            float p = std::max(throughput.x, std::max(throughput.y, throughput.z));
            if (sampler->NextFloat() > p) {
                break;
            }

            throughput *= 1 / p;
        }
    }

    m_scene->Camera->FrameBufferData.SplatPixel(x, y, color);
}

```
In English:

- Shoot a ray through the scene
- Check if we hit anything. If not we return the skybox color and break.
- Check if we hit a light. If so, we add the light emission to our color accumulation
- Choose a new direction for the next ray. We can do this uniformly, or importance sample based on the BRDF
- Evaluate the BRDF and accumulate it. Here we have to divide by the pdf of our chosen direction, in order to follow the Monte Carlo Algorithm.
- Create a new ray based on our chosen direction and where we just came from
- [Optional] Use Russian Roulette to choose if we should terminate the ray
- Goto 1

With this code, we only get color if the ray eventually hits a light. 

In addition, **it doesn't support punctual light sources, since they have no area**.

> Punctual lights are defined as parameterized, infinitely small points that emit light in well-defined directions and intensities. 

# sample the lights directly at every bounce
To fix this, we **sample the lights directly at every bounce**. 

We have to do a few small changes:

```
void RenderPixel(uint x, uint y, UniformSampler *sampler) {
    Ray ray = m_scene->Camera->CalculateRayFromPixel(x, y, sampler);

    float3 color(0.0f);
    float3 throughput(1.0f);
    SurfaceInteraction interaction;

    // Bounce the ray around the scene
    const uint maxBounces = 15;
    for (uint bounces = 0; bounces < maxBounces; ++bounces) {
        m_scene->Intersect(ray);

        // The ray missed. Return the background color
        if (ray.GeomID == INVALID_GEOMETRY_ID) {
            color += throughput * m_scene->BackgroundColor;
            break;
        }

        // Fetch the material
        Material *material = m_scene->GetMaterial(ray.GeomID);
        // The object might be emissive. If so, it will have a corresponding light
        // Otherwise, GetLight will return nullptr
        Light *light = m_scene->GetLight(ray.GeomID);

        // If this is the first bounce or if we just had a specular bounce,
        // we need to add the emmisive light
        if ((bounces == 0 || (interaction.SampledLobe & BSDFLobe::Specular) != 0) && light != nullptr) {
            color += throughput * light->Le();
        }

        interaction.Position = ray.Origin + ray.Direction * ray.TFar;
        interaction.Normal = normalize(m_scene->InterpolateNormal(ray.GeomID, ray.PrimID, ray.U, ray.V));
        interaction.OutputDirection = normalize(-ray.Direction);


        // Calculate the direct lighting
        color += throughput * SampleLights(sampler, interaction, material->bsdf, light);


        // Get the new ray direction
        // Choose the direction based on the bsdf        
        material->bsdf->Sample(interaction, sampler);
        float pdf = material->bsdf->Pdf(interaction);

        // Accumulate the weight
        throughput = throughput * material->bsdf->Eval(interaction) / pdf;

        // Shoot a new ray

        // Set the origin at the intersection point
        ray.Origin = interaction.Position;

        // Reset the other ray properties
        ray.Direction = interaction.InputDirection;
        ray.TNear = 0.001f;
        ray.TFar = infinity;


        // Russian Roulette
        if (bounces > 3) {
            float p = std::max(throughput.x, std::max(throughput.y, throughput.z));
            if (sampler->NextFloat() > p) {
                break;
            }

            throughput *= 1 / p;
        }
    }

    m_scene->Camera->FrameBufferData.SplatPixel(x, y, color);
}
```

First, we add "color += throughput * SampleLights(...)". I'll go into detail about SampleLights() in a bit. 

But, essentially, it **loops through all the lights, and returns their contribution to the color, attenuated by the BSDF**.

This is great, but we need to make one more change in order to make it correct; specifically, what happens when we hit a light. In the old code, we **added the light's emission to the color accumulation**. 

But now we **directly sample the light every bounce**, so if we added the light's emission, **we would "double dip"**. 

Therefore, the correct thing to do is... nothing; we skip accumulating the light's emission.

---

However, there are **two corner cases**:

- The first ray
- Perfectly specular bounces (aka mirrors)

If the first ray hits the light, you should see the light's emission directly. So **if we skip it, all the lights will show up as black**, even though the surfaces around them are lit.

When you hit a perfectly specular surfaces you can't directly sample a light, because an input ray has only one output. Well, technically, we could check if the input ray is going to hit a light, but there's no point; the main Path Tracing loop is going to do that anyway. Therefore, if we hit a light just after we hit a specular surface, we need to accumulate the color. **If we don't, lights will be black in mirrors**.

---

## SampleLights()
Now, let's delve into SampleLights():

```
float3 SampleLights(UniformSampler *sampler, SurfaceInteraction interaction, BSDF *bsdf, Light *hitLight) const {
    std::size_t numLights = m_scene->NumLights();

    float3 L(0.0f);
    for (uint i = 0; i < numLights; ++i) {
        Light *light = &m_scene->Lights[i];

        // Don't let a light contribute light to itself
        if (light == hitLight) {
            continue;
        }

        L = L + EstimateDirect(light, sampler, interaction, bsdf);
    }

    return L;
}
```

In English:

- Loop through all the lights
- Skip the light if we hit it
    - Don't double dip
- Accumulate the direct lighting from all the lights
- Return the direct lighting
- Finally, EstimateDirect() is just evaluating BSDF(p,ωi,ωo)Li(p,ωi)

### EstimateDirect() for punctual light

For punctual light sources, this is simple as:

```
float3 EstimateDirect(Light *light, UniformSampler *sampler, SurfaceInteraction &interaction, BSDF *bsdf) const {
    // Only sample if the BRDF is non-specular 
    if ((bsdf->SupportedLobes & ~BSDFLobe::Specular) != 0) {
        return float3(0.0f);
    }

    interaction.InputDirection = normalize(light->Origin - interaction.Position);
    return bsdf->Eval(interaction) * light->Li;
}
```

### EstimateDirect() for Area light

However, if we want lights to have area, we first need to **sample a point on the light**. Therefore, the full definition is:

```
float3 EstimateDirect(Light *light, UniformSampler *sampler, SurfaceInteraction &interaction, BSDF *bsdf) const {
    float3 directLighting = float3(0.0f);

    // Only sample if the BRDF is non-specular 
    if ((bsdf->SupportedLobes & ~BSDFLobe::Specular) != 0) {
        float pdf;
        float3 Li = light->SampleLi(sampler, m_scene, interaction, &pdf);

        // Make sure the pdf isn't zero and the radiance isn't black
        if (pdf != 0.0f && !all(Li)) {
            directLighting += bsdf->Eval(interaction) * Li / pdf;
        }
    }

    return directLighting;
}
```

We can implement light->SampleLi however we want; we can choose the point uniformly, or importance sample. In either case, **we divide the radiosity by the pdf of choosing the point**. Again, to satisfy the requirements of Monte Carlo.

# Multiple Importance Sampling
If the BRDF is highly view dependent, it may be better to choose a point based on the BRDF, instead of a random point on the light. But how do we choose? Sample based on the light, or based on the BRDF?

Why not both? Enter **Multiple Importance Sampling**. In short, we evaluate BSDF(p,ωi,ωo)Li(p,ωi)
 multiple times, using different sampling techniques, **then average them together using weights based on their pdfs**. In code this is:

```
float3 EstimateDirect(Light *light, UniformSampler *sampler, SurfaceInteraction &interaction, BSDF *bsdf) const {
    float3 directLighting = float3(0.0f);
    float3 f;
    float lightPdf, scatteringPdf;


    // Sample lighting with multiple importance sampling
    // Only sample if the BRDF is non-specular 
    if ((bsdf->SupportedLobes & ~BSDFLobe::Specular) != 0) {
        float3 Li = light->SampleLi(sampler, m_scene, interaction, &lightPdf);

        // Make sure the pdf isn't zero and the radiance isn't black
        if (lightPdf != 0.0f && !all(Li)) {
            // Calculate the brdf value
            f = bsdf->Eval(interaction);
            scatteringPdf = bsdf->Pdf(interaction);

            if (scatteringPdf != 0.0f && !all(f)) {
                float weight = PowerHeuristic(1, lightPdf, 1, scatteringPdf);
                directLighting += f * Li * weight / lightPdf;
            }
        }
    }


    // Sample brdf with multiple importance sampling
    bsdf->Sample(interaction, sampler);
    f = bsdf->Eval(interaction);
    scatteringPdf = bsdf->Pdf(interaction);
    if (scatteringPdf != 0.0f && !all(f)) {
        lightPdf = light->PdfLi(m_scene, interaction);
        if (lightPdf == 0.0f) {
            // We didn't hit anything, so ignore the brdf sample
            return directLighting;
        }

        float weight = PowerHeuristic(1, scatteringPdf, 1, lightPdf);
        float3 Li = light->Le();
        directLighting += f * Li * weight / scatteringPdf;
    }

    return directLighting;
}
```

In English:

- First, we sample the light
    - This updates interaction.InputDirection
    - Gives us the Li for the light
    - And the pdf of choosing that point on the light
- Check that the pdf is valid and the radiance is non-zero
- Evaluate the BSDF using the sampled InputDirection
- Calculate the pdf for the BSDF given the sampled InputDirection
    - Essentially, how likely is this sample, if we were to sample using the BSDF, instead of the light
- **Calculate the weight, using the light pdf and the BSDF pdf**
    - Veach and Guibas define a couple different ways to calculate the weight. **Experimentally, they found the power heuristic with a power of 2 to work the best for most cases.** I refer you to the paper for more details. The implementation is below
- Multiply the weight with the direct lighting calculation and divide by the light pdf. (For Monte Carlo) And add to the direct light accumulation.
- Then, we sample the BRDF
    - This updates interaction.InputDirection
- Evaluate the BRDF
- Get the pdf for choosing this direction based on the BRDF
- Calculate the light pdf, given the sampled InputDirection
    - This is the mirror of before. How likely is this direction, if we were to sample the light
- If lightPdf == 0.0f, then the ray missed the light, so just return the direct lighting from the light sample.
- Otherwise, calculate the weight, and add the BSDF direct lighting to the accumulation
- Finally, return the accumulated direct lighting

```
inline float PowerHeuristic(uint numf, float fPdf, uint numg, float gPdf) {
    float f = numf * fPdf;
    float g = numg * gPdf;

    return (f * f) / (f * f + g * g);
}
```

There are a number of optimizations / improvements you can do in these functions, but I've pared them down to try to make them easier to comprehend. If you would like, I can share some of these improvements.

## optimizations - Only Sampling One Light
In SampleLights() we loop through all the lights, and get their contribution. For a small number of lights, this is fine, but for hundreds or thousands of lights, this gets expensive. Fortunately, we can exploit the fact that *Monte Carlo Integration is a giant average*. Example:

Let's define

h(x)=f(x)+g(x)

Currently, we're estimating h(x)
 by:

h(x)=1N∑i=1Nf(xi)+g(xi)

But, calculating both f(x)
 and g(x)
 is expensive, so instead we do:

h(x)=1N∑i=1Nr(ζ,x)pdf

Where ζ
 is a uniform random variable, and r(ζ,x)
 is defined as:

r(ζ,x)={f(x),g(x),0.0≤ζ<0.50.5≤ζ<1.0

In this case pdf=1/2
 because the pdf must integrate to 1, and there are 2 functions to choose from.

In English:

- Randomly choose either f(x)or g(x)to evaluate.
- Divide the result by 1/2(since there are two items)
- Average

As N gets large, the estimate will converge to the correct solution.

We can apply this same principle to light sampling. Instead of sampling every light, we randomly pick one, and multiply the result by the number of lights (This is the same as dividing by the fractional pdf):

```
float3 SampleOneLight(UniformSampler *sampler, SurfaceInteraction interaction, BSDF *bsdf, Light *hitLight) const {
    std::size_t numLights = m_scene->NumLights();

    // Return black if there are no lights
    // And don't let a light contribute light to itself
    // Aka, if we hit a light
    // This is the special case where there is only 1 light
    if (numLights == 0 || numLights == 1 && hitLight != nullptr) {
        return float3(0.0f);
    }

    // Don't let a light contribute light to itself
    // Choose another one
    Light *light;
    do {
        light = m_scene->RandomOneLight(sampler);
    } while (light == hitLight);

    return numLights * EstimateDirect(light, sampler, interaction, bsdf);
}
```

In this code, all the lights have an equal chance of being picked. However, we can importance sample, if we like. For example, we can give larger lights a higher chance of being picked, or lights closer to the hit surface. You just have to divide the result by the pdf, which would no longer be 1/numLights
.

## optimizations - Multiple Importance Sampling the "New Ray" Direction
The current code only importance samples the "New Ray" direction based on the BSDF. What if we want to also importance sample based on the location of lights?

Taking from what we learned above, one method would be to shoot two "new" rays and weight each based on their pdfs. However, this is both computationally expensive, and hard to implement without recursion.

To overcome this, we can apply the same principles we learned by sampling only one light. That is, randomly choose one to sample, and divide by the pdf of choosing it.

```
// Get the new ray direction

// Randomly (uniform) choose whether to sample based on the BSDF or the Lights
float p = sampler->NextFloat();

Light *light = m_scene->RandomLight();

if (p < 0.5f) {
    // Choose the direction based on the bsdf 
    material->bsdf->Sample(interaction, sampler);
    float bsdfPdf = material->bsdf->Pdf(interaction);

    float lightPdf = light->PdfLi(m_scene, interaction);
    float weight = PowerHeuristic(1, bsdfPdf, 1, lightPdf);

    // Accumulate the throughput
    throughput = throughput * weight * material->bsdf->Eval(interaction) / bsdfPdf;

} else {
    // Choose the direction based on a light
    float lightPdf;
    light->SampleLi(sampler, m_scene, interaction, &lightPdf);

    float bsdfPdf = material->bsdf->Pdf(interaction);
    float weight = PowerHeuristic(1, lightPdf, 1, bsdfPdf);

    // Accumulate the throughput
    throughput = throughput * weight * material->bsdf->Eval(interaction) / lightPdf;
}
```

That all said, do we really want to importance sample the "New Ray" direction based on the light? For direct lighting, the radiosity is affected by both the BSDF of the surface, and the direction of the light. But for indirect lighting, the radiosity is almost exclusively defined by the BSDF of the surface hit before. So, adding light importance sampling doesn't give us anything.

Therefore, it is common to only importance sample the "New Direction" with the BSDF, but apply Multiple Importance Sampling to the direct lighting.

---

# Q&A

Thank you for the clarifying answer! I understand that if we were to use a path tracer without explicit light sampling, we would never hit a point light source. So, we can basically add its contribution. On the other hand, if we sample an area light source, we have to make sure we should not hit it again with the indirect lighting in order to avoid double dip – 

Mustafa Işık
 CommentedMay 25, 2017 at 15:37

Exactly! Is there any part that you need clarification on? Or there isn't enough detail? – 

RichieSams
 CommentedMay 25, 2017 at 15:39

Also, is multiple importance sampling used only for direct lighting calculation? Maybe I missed but I didn't see another example of it. If I shoot just one ray per bounce in my path tracer, it seems that I cannot do it for the indirect lighting calculation. – 

Mustafa Işık
 CommentedMay 25, 2017 at 15:43
3

Multiple Importance Sampling can be applied anywhere you use importance sampling. The power of multiple importance sampling is that we can combine the benefits of multiple sampling techniques. For example, in some cases, light importance sampling will be better than BSDF sampling. In other cases, vice versa. MIS will combine the best of both worlds. However, if BSDF sampling will be better 100% of the time, there is no reason to add the complexity of MIS. I added some sections to the answer to expand upon this point – 

RichieSams
 CommentedMay 25, 2017 at 18:53 
1

It seems we separated incoming radiance sources into two parts as *direct and indirect*. We *sample lights explicitly for the direct part* and while sampling this part, it is reasonable to importance sample the lights as well as BSDFs. 

For the indirect part, however, we have no idea about which direction may potentially give us higher radiance values since it is the problem itself that we want to solve. *However, we can say which direction can contribute more according to the cosine term and BSDF.* This is what I understand. Correct me if I'm wrong and thank you for your awesome answer. – 

Mustafa Işık
 CommentedMay 25, 2017 at 19:50