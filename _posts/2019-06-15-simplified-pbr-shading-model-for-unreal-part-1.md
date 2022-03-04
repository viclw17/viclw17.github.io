---
title: Simplified PBR Shading Model for Unreal Part 1
layout: post
image: 2019-06-15-simplified-pbr-shading-model-for-unreal-part-1/pbr-unreal.jpg
---

<img src="{{ site.url }}/images/2019-06-15-simplified-pbr-shading-model-for-unreal-part-1/pbr-unreal.jpg" width="640"  style="display:block; margin:auto;">
<br>
To get ready for the comming project, I am looking into an customized shading model for the base material - the one with a minimal implementation of PBR and without any unnecessary built-in features for a better rendering performance. After digging into the PBR theories as well as [this](https://blog.selfshadow.com/publications/s2013-shading-course/karis/s2013_pbs_epic_notes_v2.pdf) amazing article spacific about PBR in Unreal Engine, I finally pieced up the puzzle of the basic shading model.


# Theory Breakdown
Shading an object in rendering pipeline is basically generating colors on its surface area. In the language of physics, this is done by calculating the **outgoing light energy** (perceived by human as **colors**) of **all the points** on the surface of the object.

Light energy is described as **Radiance** with the letter $L$. (The definition of radiance is rather complex and will be noted later; it actually represents light energy differential on area and solid angle). The outgoing light contribution $L_o$ can be described as a function of a point **position** $p$ on the object surface and the **direction** $\omega_o$ of the outgoing light ray: $L_o(p,\omega_o)$.

To calculate this $L_o$ we have to integrated the incoming light energy $L_i$ (which is also a function of position and direction like $L_o$) on the hemesphere guided by surface normal at that point. $L_i$ is also weighted by the cosine of the incident angle (Lambert's Law) which can be put as $n \cdot \omega_i$.

So the total lighting phenomenon can be described as:

$$L_o(p,\omega_o) = \int\limits_{\Omega}
    	L_i(p,\omega_i) (n \cdot \omega_i)  d\omega_i$$

However, because different surface has different properties on influencing the light reaction (energy absorbing, reflection, refraction, subsurface scattering etc.) The integral is also weighted by another factor, which is the classic concept of **BRDF** (More explanation later).

In general, a typical BRDF consists a diffuse term and a specular term.

The diffuse part we are using **Lambertian Diffuse BRDF**

$$f_{Lambert} =\frac{c}{\pi}$$

The specular part we are using **Cook-Torrance Microfacet Specular BRDF**

$$f_{cook-torrance} = \frac{DFG}{4(\omega_o \cdot n)(\omega_i \cdot n)}$$

Then we also scale the diffuse term and specular term by $k_d$ and $k_s$, and put all together we get the final description of the lighting phenomenon:

$$L_o(p,\omega_o) = \int\limits_{\Omega}
    	(k_d\frac{c}{\pi} + k_s\frac{DFG}{4(\omega_o \cdot n)(\omega_i \cdot n)})
    	L_i(p,\omega_i) (n \cdot \omega_i)  d\omega_i$$

This is called **Cook-Torrance reflectance equation**. Details explained as follows.

<!-- http://graphicrants.blogspot.com/2013/08/specular-brdf-reference.html -->
<!-- <br>
<iframe width="100%" height="360" frameborder="0" src="https://www.shadertoy.com/embed/MsXBzl?gui=true&t=10&paused=true&muted=false" allowfullscreen style="display:block; margin:auto;"></iframe>
<br> -->

# Implementation

<!-- We can split the integral into 2 part to implement:

$$L_o(p,\omega_o) =
		\int\limits_{\Omega} (k_d\frac{c}{\pi}) L_i(p,\omega_i) (n \cdot \omega_i)  d\omega_i
		+
		\int\limits_{\Omega} (k_s\frac{DFG}{4(\omega_o \cdot n)(\omega_i \cdot n)})
			L_i(p,\omega_i) (n \cdot \omega_i)  d\omega_i$$ -->

Result:
<br>
<iframe width="100%" height="360" frameborder="0" src="https://www.shadertoy.com/embed/tlSGz3?gui=true&t=10&paused=true&muted=false" allowfullscreen style="display:block; margin:auto;"></iframe>
<br>

## Diffuse Term
Using Lambertian Diffuse BRDF
```c
/**
 * Standard Lambertian diffuse lighting.
 */
vec3 CalculateDiffuse(
    in vec3 albedo)
{                              
    return (albedo * ONE_OVER_PI);
}
```

## Specular Term
Using Cook-Torrance Microfacet Specular BRDF

### D -  Normal distribution function (NDF)
We are using **GGX/Trowbridge-Reitz NDF** function. Roughness lies on the range [0.0, 1.0], with lower values producing a smoother, "glossier", surface. Higher values produce a rougher surface with the specular lighting distributed over a larger surface area.

$$NDF_{GGX TR}(n, h, \alpha) = \frac{\alpha^2}{\pi((n \cdot h)^2 (\alpha^2 - 1) + 1)^2}$$

where Here h is the **halfway vector** - half-way between the light (l) and view (v); $\alpha$ is  adopted Disney’s reparameterization which equal to $Roughness^2$.

If we plot GGX/Trowbridge-Reitz NDF like [this](https://www.desmos.com/calculator/8otf8w37ke):
<iframe src="https://www.desmos.com/calculator/8otf8w37ke?embed" width="360px" height="360px" frameborder="0" style="border: 1px solid #ccc; display:block; margin:auto;"></iframe>
<br>

```c
/**
 * GGX/Trowbridge-Reitz NDF
 */
float CalculateNDF(
    in vec3  surfNorm,
    in vec3  halfVector,
    in float roughness)
{
    float a2 = (roughness * roughness);
    float halfAngle = dot(surfNorm, halfVector);

    return (a2 / (PI * pow((pow(halfAngle, 2.0) * (a2 - 1.0) + 1.0), 2.0)));
}
```

### G - Microfacet geometric attenuation
We are using **GGX/Schlick-Beckmann** function and **Smith's method** for analytical lighting scenario(TBD). The attenuation is modified by the roughness (input as $k$) and approximates the influence/amount of microfacets in the surface.

$$G_{SchlickGGX}(n, v, k)
       		 =
   		\frac{n \cdot v}
    	{(n \cdot v)(1 - k) + k }$$

Note that for analytical lighting scenario,

$$k = \frac{({Roughness} + 1)^2}{8}$$

```c
/**
 * GGX/Schlick-Beckmann microfacet geometric attenuation.
 */
float CalculateAttenuation(
    in vec3  surfNorm,
    in vec3  vector,
    in float k)
{
    float d = max(dot(surfNorm, vector), 0.0);
 	return (d / ((d * (1.0 - k)) + k));
}
```

$$G_{Smith}(n, v, l, k) = G_{l}(n, l, k) G_{v}(n, v, k)$$

```c
/**
 * GGX/Schlick-Beckmann attenuation for analytical light sources.
 */
float CalculateAttenuationAnalytical(
    in vec3  surfNorm,
    in vec3  toLight,
    in vec3  toView,
    in float roughness)
{
    float k = pow((roughness + 1.0), 2.0) * 0.125;

    // G(l) and G(v)
    float lightAtten = CalculateAttenuation(surfNorm, toLight, k);
    float viewAtten  = CalculateAttenuation(surfNorm, toView, k);

    // Smith
    return (lightAtten * viewAtten);
}
```
_For IBL (TBD)_

<!--```c
/**
 * GGX/Schlick-Beckmann attenuation for IBL light sources.
 * Uses Disney modification of k to reduce hotness.
 */
float CalculateAttenuationIBL(
    in float roughness,
    in float normDotLight,          // Clamped to [0.0, 1.0]
    in float normDotView)           // Clamped to [0.0, 1.0]
{
    float k = pow(roughness, 2.0) * 0.5;

    // G(l) and G(v)
    float lightAtten = (normDotLight / ((normDotLight * (1.0 - k)) + k));
    float viewAtten  = (normDotView / ((normDotView * (1.0 - k)) + k));

    // Smith
    return (lightAtten * viewAtten);
}
``` -->

### F - Fresnel reflectivity
We are using **Fresnel-Schlick approximation** and **Spherical Gaussian approximation**. The **Metallic** parameter controls the fresnel incident value (fresnel0), more explanation later.
```c
/**
 * Calculates the Fresnel reflectivity.
 */
vec3 CalculateFresnel(
    in vec3 surfNorm,
    in vec3 toView,
    in vec3 fresnel0)
{
	float d = max(dot(surfNorm, toView), 0.0);
    float p = ((-5.55473 * d) - 6.98316) * d;

    // Fresnel-Schlick approximation
    return fresnel0 + ((1.0 - fresnel0) * pow(1.0 - d, 5.0));
    // modified by Spherical Gaussian approximation to replace the power, more efficient
    return fresnel0 + ((1.0 - fresnel0) * pow(2.0, p));
}
```

### Put together
```c
/**
 * Cook-Torrance BRDF for analytical light sources.
 */
vec3 CalculateSpecularAnalytical(
    in    vec3  surfNorm,            // Surface normal
    in    vec3  toLight,             // Normalized vector pointing to light source
    in    vec3  toView,              // Normalized vector point to the view/camera
    in    vec3  fresnel0,            // Fresnel incidence value
    inout vec3  sfresnel,            // Final fresnel value used a kS
    in    float roughness)           // Roughness parameter (microfacet contribution)
{
    vec3 halfVector = CalculateHalfVector(toLight, toView);

    float ndf      = CalculateNDF(surfNorm, halfVector, roughness);
    float geoAtten = CalculateAttenuationAnalytical(surfNorm, toLight, toView, roughness);

    sfresnel = CalculateFresnel(surfNorm, toView, fresnel0);

    vec3  numerator   = (sfresnel * ndf * geoAtten); // FDG
    float denominator = 4.0 * dot(surfNorm, toLight) * dot(surfNorm, toView);

    return (numerator / denominator);
}
```
_For IBL (TBD)_

<!-- Importance Sampling + Monte Carlo

<br>
<iframe width="100%" height="360" frameborder="0" src="https://www.shadertoy.com/embed/4lscWj?gui=true&t=10&paused=true&muted=false" allowfullscreen style="display:block; margin:auto;"></iframe>
<br>
http://holger.dammertz.org/stuff/notes_HammersleyOnHemisphere.html

```c
/**
 * Generates a 2D directional vector on the hemisphere from the Hammersley point set.
 * Source: http://holger.dammertz.org/stuff/notes_HammersleyOnHemisphere.html
 */
vec2 Hammersley(float i, float numSamples)
{   
    uint bits = uint(i);

    bits = (bits << 16u) | (bits >> 16u);
    bits = ((bits & 0x55555555u) << 1u) | ((bits & 0xAAAAAAAAu) >> 1u);
    bits = ((bits & 0x33333333u) << 2u) | ((bits & 0xCCCCCCCCu) >> 2u);
    bits = ((bits & 0x0F0F0F0Fu) << 4u) | ((bits & 0xF0F0F0F0u) >> 4u);
    bits = ((bits & 0x00FF00FFu) << 8u) | ((bits & 0xFF00FF00u) >> 8u);

    float radicalInverseVDC = float(bits) * 2.3283064365386963e-10; // / 0x100000000

    return vec2((i / numSamples), radicalInverseVDC);
}

/**
 * Importance Sampling to solve the radiance integral.
 *
 * We use importance sampling (as opposed to uniform or random (Monte Carlo)) to
 * generate light sample vectors that are biased to the microsurface halfway
 * vector based on the roughness.
 */
vec3 ImportanceSample(
    in vec2  xi,
    in float roughness,
    in vec3  surfNorm)
{
	float a = (roughness * roughness);

    // Spherical Coordinates to Cartesian
    float phi = 2.0 * PI * xi.x;
    float cosTheta = sqrt((1.0 - xi.y) / (1.0 + (a * a - 1.0) * xi.y));
    float sinTheta = sqrt(1.0 - (cosTheta * cosTheta));

    vec3 H = vec3((sinTheta * cos(phi)), (sinTheta * sin(phi)), cosTheta);

    // From Tangent-Space to World-Space
    vec3 upVector = StepValue3(0.999, surfNorm.z, vec3(1.0, 0.0, 0.0), vec3(0.0, 0.0, 1.0));
    vec3 tangentX = normalize(cross(upVector, surfNorm));
    vec3 tangentY = cross(surfNorm, tangentX);

    return ((tangentX * H.x) + (tangentY * H.y) + (surfNorm * H.z));
}

/**
 * Performs the Riemann Sum approximation of the IBL lighting integral.
 *
 * The ambient IBL source hits the surface from all angles. We average
 * the lighting contribution from a number of random light directional
 * vectors to approximate the total specular lighting.
 *
 * The number of steps is controlled by the 'IBL Steps' global.
 */
vec3 CalculateSpecularIBL(
    in    vec3  surfNorm,
    in    vec3  toView,
    in    vec3  fresnel0,
    inout vec3  sfresnel,
    in    float roughness)
{
    vec3 totalSpec = vec3(0.0);
    vec3 toSurfaceCenter = reflect(-toView, surfNorm);

    for(float i = 0.0; i < IBLSteps; ++i)
    {
        // The 2D hemispherical sampling vector
    	vec2 xi = Hammersley(i, IBLSteps);

        // Bias the Hammersley vector towards the specular lobe of the surface roughness
        vec3 H = ImportanceSample(xi, roughness, surfNorm);

        // The light sample vector
        vec3 L = (2.0 * dot(toView, H) * H) - toView;

        float NoV = clamp(dot(surfNorm, toView), 0.0, 1.0);
        float NoL = clamp(dot(surfNorm, L), 0.0, 1.0);
        float NoH = clamp(dot(surfNorm, H), 0.0, 1.0);
        float VoH = clamp(dot(toView, H), 0.0, 1.0);

        if(NoL > 0.0)
        {
            vec3 color = SampleEnvironment(L);

            float geoAtten = CalculateAttenuationIBL(roughness, NoL, NoV);
            vec3  fresnel = CalculateFresnel(surfNorm, toView, fresnel0);

            sfresnel += fresnel;
#ifdef SPECULAR_NDF_ONLY
            totalSpec += 0.0;
#elif defined(SPECULAR_ATTEN_ONLY)
            totalSpec += geoAtten / (NoH * NoV);
#elif defined(SPECULAR_FRESNEL_ONLY)
            totalSpec += fresnel / (NoH * NoV);
#else
            totalSpec += (color * fresnel * geoAtten * VoH) / (NoH * NoV);
#endif
        }
    }

    sfresnel /= IBLSteps;

    return (totalSpec / IBLSteps);
}
``` -->

# Solve Reflectance Equation
Now we have to combine Diffuse Term + Specular Term.

```c
/**
 * Calculates the total light contribution for the analytical light source.
 */
vec3 CalculateLightingAnalytical(
    in vec3  surfNorm,
    in vec3  toLight,
    in vec3  toView,
    in vec3  albedo,
    in float roughness)
{
    vec3 fresnel0 = mix(vec3(0.04), albedo, Metallic);
    vec3 ks       = vec3(0.0);
    vec3 kd       = (1.0 - ks);
    vec3 diffuse  = CalculateDiffuse(albedo);
    vec3 specular = CalculateSpecularAnalytical(surfNorm, toLight, toView, fresnel0, ks, roughness);

    float angle = clamp(dot(surfNorm, toLight), 0.0, 1.0);

    return ((kd * diffuse) + specular) * angle;
}
```
_For IBL (TBD)_

<!-- ```c
/**
 * Calculates the total light contribution from the ambient IBL environmental map.
 */
vec3 CalculateLightingIBL(
    in vec3  surfNorm,
    in vec3  toView,
    in vec3  albedo,
    in float roughness)
{
    vec3 fresnel0 = mix(vec3(0.04), albedo, Metallic);
    vec3 ks       = vec3(0.0);
    vec3 diffuse  = CalculateDiffuse(albedo);
    vec3 specular = CalculateSpecularIBL(surfNorm, toView, fresnel0, ks, roughness);
    vec3 kd       = (1.0 - ks);

#ifdef DIFFUSE_ONLY
	return diffuse;
#elif defined(SPECULAR_ONLY) || defined(SPECULAR_NDF_ONLY) || defined(SPECULAR_ATTEN_ONLY) || defined(SPECULAR_FRESNEL_ONLY)
    return specular;
#else
    return ((kd * diffuse) + specular);
#endif
}
``` -->

TBC - IBL/Unreal Material Implementation

# Reference
1. [Learnopengl.com - PBR - Theory](https://learnopengl.com/PBR/Theory)
2. [Real Shading in Unreal Engine 4](https://blog.selfshadow.com/publications/s2013-shading-course/karis/s2013_pbs_epic_notes_v2.pdf)
3. [Shadertoy - PBR Lighting Demo](https://www.shadertoy.com/view/MsXBzl)
4. [Specular BRDF Reference](http://graphicrants.blogspot.com/2013/08/specular-brdf-reference.html)
5. [PBRT](http://www.pbr-book.org/)
