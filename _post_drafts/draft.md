# TU Wien notes

## 01_light

- we have to sum up all the light. yes that is an **integral**.
- we have to sum up from all direction, that is a 
**hemisphere**

---

$$L_i(x) = \int_\Omega L_i(x,w)cos(\theta_x)dw$$

light arriving at point x

light from direction w - **by ray tracing**

differential solid angle dw

> irradiance

---

solid angle: projected area on unit sphere. full solid angle is 4pi

---

relationship between a surface patch and the solid angle

$$dw = \frac{dAcos\theta}{r^2}$$

---

$$L_i^{[l]}(x) = \int_{S_l} L_e^{[l]}(y) cos(\theta_x) \frac{cos\theta_y}{r^2} dA_y$$

light from source [l] arriving at point x

light intensity at position y on the surface

$$\frac{cos\theta_y}{r^2} dA_y = dw$$

---

radiance L = flux per unit projected area per unit solid angle

$$L = \frac{d\Phi}{dA^\perp dw}$$

radiance is a density over both space and angle.

---

$$L_e(x,v) = \int_\Omega f_r(x, w \rightarrow v) L_i(x,w) cos(\theta_x) dw$$

light going in direction v, viewing

material modelled by BRDF

> how much light is reflected from a **given direction** into **another given direction** at a **given position**, and in which wavelengths

light from direction w 

differential solid angle dw

---

white furnace test, energy conservation

(set $L_i$ to 1 and check $L_e \leq 1$)

... we can derive: f_r or a white diffuse material is 1/pi




## 03_rendering equation
Photons are emitted from light sources, reflected by surfaces in the scene until they reach the sensor. In rendering, we (can) go the opposite way. We trace importons until they reach a light source.

---

recap light integral: compute the light which is going into direction v, integrate over hemisphere, check all directions for incoming light, cosine weighting anf material.

---

### 1 recursive formulation

$$L_e(x,v) = E(x,v) + \int_{\Omega} f_r(x,w \rightarrow v) L_i(x,w) cos(\theta_x) dw$$

---

$$L(x_1 \rightarrow v) = E(x_1 \rightarrow v) + \int_{\Omega_1} f_r(x_1,w_1 \rightarrow v) L(x_1 \leftarrow w_1) cos(\theta_x) dw_1$$

$$L(x_1 \rightarrow w_2) = E(x_2 \rightarrow w_2) + \int_{\Omega_2} f_r(x_2,w \rightarrow w_2) L(x_2 \leftarrow w) cos(\theta_x) dw$$

$L(x_1 \leftarrow w_1) = L(x_1 \rightarrow w_2)$

$$L(x \rightarrow v) = E(x \rightarrow v) + \int_{\Omega} f_r(x,w \rightarrow v) L(x \leftarrow w) cos(\theta_x) dw$$
---

### 2 operator formulation

$$L = L_e + TL$$

T: light transport operator

$$T = KG$$

K: local scattering operator, $L_o = KL_i$, turn incoming radiance into outgoing radiance, material

G: propagation operator, $L_i = GL_o$, turn outgoing radiance into incoming radiance, ray-tracing

$$L = (I-T)^{-1} L_e$$

$S = (I-T)^{-1}$ solution operator

$$S = (I-T)^{-1} = I + T +T^2+...$$

$$L = E + TE+T^2E+..., |T^k|\leq 1$$

This equation reaches an equilibrium after infinite time / iterations, after which it gives us the solution for the light distribution in the scene.

---

### 3 path integral formulation
So the path integral formulation is really just an integral which integrates over all surfaces at the same time

---

https://computergraphics.stackexchange.com/questions/9015/rendering-equation-in-terms-of-paths-rather-than-directions

---

https://pbr-book.org/3ed-2018/Light_Transport_III_Bidirectional_Methods/The_Path-Space_Measurement_Equation#

> 16.1 The Path-Space Measurement Equation
In light of the path integral form of the LTE from Equation (14.16), it’s useful to go back and formally describe the quantity that is being estimated when we compute pixel values for an image. Not only does this let us see how to apply the LTE to a wider set of problems than just computing 2D images (e.g., to precomputing scattered radiance distributions at the vertices of a polygonal model), but this process also leads us to a key theoretical mechanism for understanding the bidirectional path tracing and photon mapping algorithms in this chapter. 

https://pbr-book.org/3ed-2018/Light_Transport_I_Surface_Reflection/The_Light_Transport_Equation#TheSurfaceFormoftheLTE

https://pbr-book.org/3ed-2018/Light_Transport_I_Surface_Reflection/The_Light_Transport_Equation#IntegraloverPaths

https://pbr-book.org/3ed-2018/Light_Transport_III_Bidirectional_Methods/The_Path-Space_Measurement_Equation#SamplingCameras

---

$$I_j = \int_\Omega f_j(\bar{x}) d_\mu(\bar{x})$$

$$\bar{x} = x_0 x_1...x_k$$

$$d_\mu(\bar{x}) = dA(x_0) dA(x_1) ... dA(x_k)$$

$$f_j(\bar{x}) = L_e(x_0 \rightarrow x_1)G(x_0 \rightarrow x_1)f_s(x_0 \rightarrow x_1 \rightarrow x_2)...W_e^{(j)}(x_{k-1} \rightarrow x_k)$$

$$G(x \leftrightarrow x') = V(x \leftrightarrow x') \frac{|cos(\theta_o)cos(\theta_i')|}{||x-x'||^2}$$

$f_j$ is a product of several factors:
- the light emission $L_e$, which is simply the brightness of the light at position $x_0$ 
- geometry factors between each pair of vertices -- $G$
- the scattering factors $f_s$ for each inner vertex (reflection point), which model the material
- and finally the importance emission from the camera $W_e$.




## 04_path tracing
https://pbr-book.org/4ed/Light_Transport_I_Surface_Reflection/A_Simple_Path_Tracer

### path tracing roadmap
- rendering equation recap
- direct lighting
- path tracing v0.5
- sample distribution
- russian roulette
- bsdf interface
- path tracing v1.0

> drand48 is a Linux function, not a standard C++ function. 

### Direct lighting with RE

$$L(x \rightarrow v) = E_x + \int_\Omega \frac{1}{\pi} E_y \; cos(\theta_\omega)dw$$

### Indirect lightng with RE

$$L(x \rightarrow v) = E_x + \int_\Omega f_r \; \left( E_{x'} + \int_{\Omega'} f_{r'} \; ... \; cos(\theta_{\omega'})dw' \right) \; cos(\theta_\omega)dw$$

### Sample Distribution

$$L(x \rightarrow v) = E_x + \int_\Omega f_r \; \left( E_{x'} + \int_{\Omega'} f_{r'} \; ... \; cos(\theta_{\omega'})dw' \right) \; cos(\theta_\omega)dw$$

rewrite this one big integral:


$$L(x \rightarrow v) = E_x$$

$$+ \int_\Omega f_r \; E_{x'} \; cos(\theta_\omega)dw$$

$$+ \int_\Omega f_r \; \int_{\Omega'} f_r' \; E_{x''} \; cos(\theta_{\omega'})cos(\theta_w) \; dw'dw$$

$$ + ... $$

---

The **path integral form** used **a single integral for each bounce**!

$$ L(x \rightarrow v) = E_x $$

$$+ \int_{\Omega_1} \; f_r \; E_{x'} \; cos(\theta_\omega) \; d\mu({\bar x})$$

$$+ \int_{\Omega_2} \; f_r f_r' \; E_{x''} \; cos(\theta_{\omega'})cos(\theta_w) \; d\mu({\bar x})$$

$$ + ... $$

> https://www.overleaf.com/learn/latex/Integrals%2C_sums_and_limits

replace each integral with **Monte Carlo integration**

$$ L(x \rightarrow v) = E_x $$

$$+ \frac{1}{N} \sum_{i=1}^{N} \; f_r \; E_{x'} \; cos(\theta_\omega) \; \frac{1}{p(w)}$$

$$+ \frac{1}{N} \sum_{i=1}^{N} \; f_r f_r' \; E_{x''} \; cos(\theta_{\omega'})cos(\theta_w) \; \frac{1}{p(w)p(w')}$$

$$ + ... $$

 pull the sum to the front...

---

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