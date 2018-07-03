---
title: Raytracer Case Study - Part 3 - Radiance
date: 2018-07-02
tags:
- Computer Graphics
- Raytracing
- PBR
---
# Radiance
## Steradian
<img src="https://upload.wikimedia.org/wikipedia/commons/9/98/Steradian.svg" width="160"  style="display:block; margin:auto;">
<br>

A graphical representation of 1 [steradian](https://en.wikipedia.org/wiki/Steradian).
The sphere has radius $r$, and in this case the area $A$ of the **red highlighted surface patch** is $r^2$. The solid angle $Ω$ equals [$A/r^2$] $sr$ which is 1$sr$ in this example. The entire sphere has a solid angle of $4πsr$.


>_The steradian, like the radian, is a **dimensionless unit**, essentially because a solid angle is the ratio between the area subtended and the square of its distance from the center: both the numerator and denominator of this ratio have dimension length squared (i.e. $L^2/L^2 = 1$, so, dimensionless). It is useful, however, to distinguish between dimensionless quantities of a different nature, so the symbol "$sr$" is used to indicate a solid angle. For example, **radiant intensity** can be measured in **watts per steradian** ($W/sr$)._

The solid angle is related to the area it cuts out of a sphere:

$$\Omega ={\frac {A}{r^{2}}}\ \mathrm {sr} \,={\frac {2\pi h}{r}}\ \mathrm {sr}$$

where
- $A$ is the surface area of the spherical cap, $2\pi rh$, (where $h$ stands for the "height" of the cap)
- $r$ is the radius of the sphere
- $sr$ is the unit, steradian.

Since $A = r^2$, it corresponds to the area of a spherical cap $A = 2πrh$, so the relationship $h/r= 1/2π$ holds.

<img src="https://upload.wikimedia.org/wikipedia/commons/2/20/Steradian_cone_and_cap.svg" width="240"  style="display:block; margin:auto;">
<br>

Therefore, one steradian corresponds to the plane (i.e. radian) angle of the cross-section of a simple cone subtending the plane angle $2θ$, with $θ$ given by:

$$\theta =\arccos ({\frac {r-h}{r}}) \,=\arccos (1-{\frac {h}{r}})\,=\arccos (1-{\frac {1}{2\pi }})\approx 0.572\,{\text{ rad,}}{\text{ or }}32.77^{\circ }$$

and $2θ ≈ 1.144{\text{ rad,}}{\text{ or }}65.54^{\circ }$ is the plane aperture angle.

The solid angle of a cone whose cross-section subtends the angle $2θ$ is:

$$ \Omega = 2\pi\left(1 - \cos\theta\right)\,\mathrm{sr}$$

## Radiance
In radiometry, radiance is the radiant flux emitted, reflected, transmitted or received by a given surface, per unit solid angle per unit projected area.

These are **directional quantities**. The SI unit of radiance is the watt per steradian per square metre ($W·1/sr·1/m^2$). Radiance is used to characterize **diffuse emission and reflection** of electromagnetic radiation. Historically, radiance is called "**intensity**".

Radiance of a surface, denoted $L_{e,Ω}$ ("$e$" for "energetic", to avoid confusion with photometric quantities, and "$Ω$" to indicate this is a directional quantity), is defined as

$$L_{{{\mathrm  {e}},\Omega }}={\frac  {\partial ^{2}\Phi _{{\mathrm  {e}}}}{\partial \Omega \,\partial A\cos \theta }}$$

where
- $∂$ - is the partial derivative symbol;
- $Φ_e$ - is the radiant flux emitted, reflected, transmitted or received;
- $Ω$ - is the solid angle ($sr$);
- $Acosθ$ - is the projected area.

Radiance is useful because it indicates how much of the power **emitted, reflected, transmitted or received** by a surface will be received by an optical system looking at that surface **from a specified angle** of view.

In this case, the solid angle of interest is the solid angle subtended by the optical system's entrance pupil. Since the eye is an optical system, radiance and its cousin **luminance** are good indicators of how bright an object will appear. For this reason, radiance and luminance are both sometimes called "**brightness**"(now discouraged).

The radiance divided by the index of refraction squared is invariant in geometric optics. This means that for an ideal optical system in air, the radiance at the output is the same as the input radiance. This is sometimes called **conservation of radiance**. For real, passive, optical systems, the output radiance is at most equal to the input, unless the index of refraction changes. As an example, if you form a demagnified image with a lens, the optical power is concentrated into a smaller area, so the irradiance is higher at the image. The light at the image plane, however, fills a larger solid angle so the radiance comes out to be the same assuming there is no loss at the lens.

In general $L_{e,Ω}$ is a function of **viewing direction**, depending on $θ$ through $cos θ$ and azimuth angle through $∂Φe/∂Ω$. For the special case of a **Lambertian surface**, $∂^2Φe/(∂Ω ∂A)$ is proportional to $cos θ$, and $L_{e,Ω}$ is isotropic (independent of viewing direction).

When calculating the radiance emitted by a source, $A$ refers to an area on the surface of the source, and $Ω$ to the solid angle into which the light is emitted.

When calculating radiance received by a detector, $A$ refers to an area on the surface of the detector and $Ω$ to the solid angle subtended by the source as viewed from that detector. When radiance is conserved, the radiance emitted by a source is the same as that received by a detector observing it.
