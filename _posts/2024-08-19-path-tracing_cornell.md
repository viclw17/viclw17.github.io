---
title: "Path Tracing Cornell Note"
type: article
layout: post
# image: 2024-08-15-pbr-math-dump/thomas-t-OPpCbAAKWv8-unsplash.jpg
---
Archived source from [link](https://www.cs.cornell.edu/courses/cs6630/2022fa/)

* This will become a table of contents (this text will be scrapped).
{:toc}

# Version 0: brute force recursive sampling

The idea of path tracing is to use Monte Carlo to compute the illumination integral for a surface point, but to make a recursive call to get all radiance incident on the surface rather than just the direct radiance.

$$L_r(x,w)=\int_{H^2}f_r(x,w,w')L_i(x,w')d\mu(w')$$

- integrand: $f_r(x,w,w')L_i(x,w')$
- probability: $d\mu(w')$, uniform $\frac{1}{\pi}$
- estimator:

$$g = f/p = f_r(x,w,w')L_i(x,w') / \frac{1}{\pi}$$

```c
// estimator
rayRadianceEst(x, ω): 
    y = traceRay(x, ω) 
    return emittedRadiance(y, –ω) + reflectedRadianceEst(y, –ω) // recursive

// estimator
reflectedRadianceEst(x, ωr): 
    ωi = uniformRandomPSA(n(x)) 
    return π * brdf(x, ωi, ωr) * rayRadianceEst(x, ωi)

```



# Version 0.5: Russian Roulette

When we are evaluating the integral we 
- replace a fraction of the samples with zero (i.e. terminate some paths) and 
- increase the weight of the remaining samples to preserve the mean.

```c
rayRadianceEst(x, ω): 
    y = traceRay(x, ω) 
    return emittedRadiance(y, –ω) + reflectedRadianceEst(y, –ω) 

reflectedRadianceEst(x, ωr): 
    if random() < survivalProbability: 
        ωi = uniformRandomPSA(n(x)) 
        return π * brdf(x, ωi, ωr) * rayRadianceEst(x, ωi) / survivalProbability 
    else 
        return 0
```

# Version 0.75: BRDF sampling

We can improve things by doing **importance sampling according to the BRDF** rather than the **uniform projected solid angle** sampling

```c
rayRadianceEst(x, ω): 
    y = traceRay(x, ω) 
    return emittedRadiance(y, –ω) + reflectedRadianceEst(y, –ω) 

reflectedRadianceEst(x, ωr): 
    if random() < survivalProbability: 
        ωi, pdf = brdfSample(x, n(x)) 
        return brdf(x, ωi, ωr) * rayRadianceEst(x, ωi) / (pdf * survivalProbability) 
    else 
        return 0

```

# Version 1.0: direct illumination

Separate the integral into direct and indirect and use two samples

$$L_r(x,w)=\int_{H^2}f_r(x,w,w') \; [L_i^{0}(x,w'), L_i^{+}(x,w')] \; d\mu(w')$$

$$=\int_{H^2}f_r(x,w,w') \; L_i^{0}(x,w')d\mu(w') + \int_{H^2}f_r(x,w,w') \; L_i^{+}(x,w') \; d\mu(w')$$


- sample according to luminaires $P_L$
- sample according to BRDF $P_B$

This means we trace two rays, 

- one by luminaire (L) sampling and 
- one by BRDF (B) sampling.

The L ray goes toward a luminaire and its **radiance value is the emitted light** from its direction.

> We don’t recurse on the L ray (called a **shadow ray**).  

The B ray (the indirect ray) goes in some arbitrary direction (maybe toward a luminaire, maybe not) but in either case its **radiance value is the reflected light (recursively estimated)** of the surface it hits.  

> We don’t include emission in the B rays.  

In the example code on the slide, this is done by having the caller trace the ray and then call `reflectedRadianceEst` (rather than `rayRadianceEst`, which would have included emitted light)

```c
rayRadianceEst(x, ω): 
    y = traceRay(x, ω) 
    return emittedRadiance(y, –ω) + reflectedRadianceEst(y, –ω) 

reflectedRadianceEst(x, ωr): 
    return directRadianceEst(x, ωr) + indirectRadianceEst(x, ωr) 

// L
directRadianceEst(x, ωr): 
    ωi, pdf = luminaireSample(x, n(x)) 
    y = traceRay(x, ωi) 
    return brdf(x, ωi, ωr) * emittedRadiance(y, –ωi) / pdf

// B
indirectRadianceEst(x, ωr): 
    if random() < survivalProbability: 
        ωi, pdf = brdfSample(x, n(x)) 
        y = traceRay(x, ωi) 
        return brdf(x, ωi, ωr)  * reflectedRadianceEst(y, –ωi) / (pdf * survivalProbability) 
    else: 
        return 0

```

# Version 1.0m: direct by multiple importance
We got the best (or at least most robust) results for direct illumination by using **multiple importance sampling to combine luminaire and BRDF sampling**.

```c
directRadianceEst(x, ωr): 
    ωl, pll = luminaireSample(x, n(x)) 
    pbl = brdfPDF(ωl) 
    ωb, pbb = brdfSample(x, n(x)) 
    plb = luminairePDF(ωb) 

    yl = traceRay(x, ωl) 
    yb = traceRay(x, ωb) 

    fl = brdf(x, ωl, ωr)  * emittedRadiance(yl, –ωi) 
    fb = brdf(x, ωb, ωr)  * emittedRadiance(yb, –ωi) 

    return fl / (pll + pbl) + fb / (plb + pbb)
```

# Version 1.1: sharing the BRDF ray
For each reflection it generates three rays by the time it is done: 
- an L and a B for direct, and then 
- later another B for indirect

This is wasteful, because **those two samples don’t need to be independent**. 

They are not samples of the same estimator; they are samples contributing to two estimators we are adding together. 

So we can save work by **tracing a single B ray and using it to sample both emitted and reflected light.**  

The weighting is important: the contribution of emitted light is weighted against the luminaire sample using **Veach & Guibas’s balance heuristic**, whereas the contribution of reflected light is just normalized as its own separate estimate and added in.

Doing this in the pseudocode results in a monolithic `reflectedRadianceEst` function that is perhaps 
harder to read, but performs well.

```c
reflectedRadianceEst(x, ωr): 
    ωl, pll = luminaireSample(x, n(x)) 
    pbl = brdfPDF(ωl) 
    ωb, pbb = brdfSample(x, n(x)) 
    plb = luminairePDF(ωb) 

    yl = traceRay(x, ωl) 
    yb = traceRay(x, ωb) 

    fl = brdf(x, ωl, ωr) * emittedRadiance(yl, –ωl) 
    fb = brdf(x, ωb, ωr) * emittedRadiance(yb, –ωb) 

    reflRad = fl / (pll + pbl) + fb / (plb + pbb) 

    if random() < survivalProbability: 
        reflRad += brdf(x, ωb, ωr) / pbb * reflectedRadianceEst(yb, –ωb) / survivalProbability 
    
    return reflRad
```

<!-- Note that none of these methods will do a good job of sampling paths that undergo specular 
transport between a small light source and a diffuse surface (that is, “caustic” paths). -->