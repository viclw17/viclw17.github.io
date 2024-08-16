# TU Wien notes

### RR

**Monte Carlo is all about picking samples and then compensating**

---

### Improvement

Economize on samples –squeeze out whatever we can
- Better sampling strategies (importance sampling)
- Exploiting light source sampling (next-event estimation)
- Combining sampling strategies (multiple importance sampling)

Improving our scene intersection tests
- Build spatial acceleration structures
- Optimized traversal strategies

Support spectacular specular, glossy and transparent materials



---



# pbrt code notes
## RayIntegrator::EvaluatePixelSample
```c
void RayIntegrator::EvaluatePixelSample

    // Initialize _CameraSample_ for current sample
    Filter filter = camera.GetFilm().GetFilter();
    CameraSample cameraSample = GetCameraSample(sampler, pPixel, filter);

    // Generate camera ray for current sample
    pstd::optional<CameraRayDifferential> cameraRay = camera.GenerateRayDifferential(cameraSample, lambda);


    SampledSpectrum L(0.);
    // Evaluate radiance along camera ray
    L = cameraRay->weight * Li(...)

    // Add camera ray's contribution to image
    camera.GetFilm().AddSample(pPixel, L, lambda, &visibleSurface, cameraSample.filterWeight);
```

## details
```c
SampledSpectrum SimplePathIntegrator::Li(RayDifferential ray, SampledWavelengths &lambda,
                                         Sampler sampler, ScratchBuffer &scratchBuffer,
                                         VisibleSurface *) const {
    // Estimate radiance along ray using simple path tracing
    SampledSpectrum L(0.f), beta(1.f);

    bool specularBounce = true;
    int depth = 0;

    // while loop!!!
    while (beta) {
        // Find next _SimplePathIntegrator_ vertex and accumulate contribution
        // Intersect _ray_ with scene
        pstd::optional<ShapeIntersection> si = Intersect(ray);

        // Account for infinite lights if ray has no intersection
        if (!si) {
            if (!sampleLights || specularBounce)
                for (const auto &light : infiniteLights)
                    L += beta * light.Le(ray, lambda);
            break;
        }

        // Account for emissive surface if light was not sampled
        SurfaceInteraction &isect = si->intr;
        if (!sampleLights || specularBounce)
            L += beta * isect.Le(-ray.d, lambda);

        // End path if maximum depth reached
        if (depth++ == maxDepth)
            break;

        // Get BSDF and skip over medium boundaries
        BSDF bsdf = isect.GetBSDF(ray, lambda, camera, scratchBuffer, sampler);
        if (!bsdf) {
            specularBounce = true;
            isect.SkipIntersection(&ray, si->tHit);
            continue;
        }

        // Sample direct illumination if _sampleLights_ is true
        Vector3f wo = -ray.d;
        if (sampleLights) {
            pstd::optional<SampledLight> sampledLight =
                lightSampler.Sample(sampler.Get1D());

            if (sampledLight) {
                // Sample point on _sampledLight_ to estimate direct illumination
                Point2f uLight = sampler.Get2D();
                pstd::optional<LightLiSample> ls =
                    sampledLight->light.SampleLi(isect, uLight, lambda);
                if (ls && ls->L && ls->pdf > 0) {
                    // Evaluate BSDF for light and possibly add scattered radiance
                    Vector3f wi = ls->wi;
                    SampledSpectrum f = bsdf.f(wo, wi) * AbsDot(wi, isect.shading.n);
                    if (f && Unoccluded(isect, ls->pLight))
                        L += beta * f * ls->L / (sampledLight->p * ls->pdf);
                }
            }
        }

        // Sample outgoing direction at intersection to continue path
        if (sampleBSDF) {
            // Sample BSDF for new path direction
            Float u = sampler.Get1D();
            pstd::optional<BSDFSample> bs = bsdf.Sample_f(wo, u, sampler.Get2D());
            if (!bs)
                break;
            beta *= bs->f * AbsDot(bs->wi, isect.shading.n) / bs->pdf;
            specularBounce = bs->IsSpecular();
            ray = isect.SpawnRay(bs->wi);

        } else {
            // Uniformly sample sphere or hemisphere to get new path direction
            Float pdf;
            Vector3f wi;
            BxDFFlags flags = bsdf.Flags();
            if (IsReflective(flags) && IsTransmissive(flags)) {
                wi = SampleUniformSphere(sampler.Get2D());
                pdf = UniformSpherePDF();
            } else {
                wi = SampleUniformHemisphere(sampler.Get2D());
                pdf = UniformHemispherePDF();
                if (IsReflective(flags) && Dot(wo, isect.n) * Dot(wi, isect.n) < 0)
                    wi = -wi;
                else if (IsTransmissive(flags) && Dot(wo, isect.n) * Dot(wi, isect.n) > 0)
                    wi = -wi;
            }
            beta *= bsdf.f(wo, wi) * AbsDot(wi, isect.shading.n) / pdf;
            specularBounce = false;
            ray = isect.SpawnRay(wi);
        }

        CHECK_GE(beta.y(lambda), 0.f);
        DCHECK(!IsInf(beta.y(lambda)));
    }
    return L;
}

```

## Integrator classes
```c
class ImageTileIntegrator : public Integrator
    void Render();

    // pure
    virtual void EvaluatePixelSample(Point2i pPixel, 
                                    int sampleIndex, 
                                    Sampler sampler, 
                                    ScratchBuffer &scratchBuffer) = 0;


class RayIntegrator : public ImageTileIntegrator
    // real
    void EvaluatePixelSample

    // pure
    virtual SampledSpectrum Li(RayDifferential ray, 
                            SampledWavelengths &lambda, 
                            Sampler sampler, 
                            ScratchBuffer &scratchBuffer, 
                            VisibleSurface *visibleSurface)  const = 0;


class SimplePathIntegrator : public RayIntegrator
    // real
    SampledSpectrum Li

    static std::unique_ptr<SimplePathIntegrator> Create(...)

```

# notes

https://fruty.io/2018/03/05/the-rendering-equation-explained/

Numerical Solution

This equation belongs to a wider category of equations, called “**Fredholm equations of the second kind**”. So far scientists have found analytical solutions to several subcategories of Fredholm equations of the second kind, unfortunately the rendering equation is not one of them, one of the reasons being that the equation is “**infinitely recursive**”: the energy coming from a ray depends on where and how the ray bounced before.

The rendering equation includes an **integral over surface patches around the point of interest**. The equation can be **re-written as an integral over light paths**. Solving it is then all about deciding which light paths you can ignore, or compute differently, to speed up the process. The integral is **highly dimensional (the space of light paths is very large)**, and for such problems **Monte Carlo** methods are a good fit.

Most renderers today (Arnold etc) use some form of Monte Carlo ray tracing. It is important to understand that Monte Carlo methods are **statistical methods which involve random sampling**, of light paths. This means a lot of **random memory access**, which in turns explains why a lot of commercial renderers perform better on **CPUs** (vs GPUs).

However, as always, it is always possible to implement an algorithm in many ways, and some renderers (ex: Octane Render) implement Monte Carlo ray tracing in a GPU-friendly way.


