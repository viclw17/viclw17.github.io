---
title: Raytracing - Rendering Equation
date: 2018-07-08
tags:
- Computer Graphics
- Raytracing
- Math
- PBR
---
<img src="/images/pbr-equation.jpg" width="640"  style="display:block; margin:auto;">

# The Rendering Equation
The physical basis for the rendering equation is the **law of conservation of energy**. Assuming that L denotes radiance, we have that at each particular position and direction, the outgoing light (Lo) is the sum of the emitted light (Le) and the reflected light. The reflected light itself is the sum from all directions of the incoming light (Li) multiplied by the surface reflection and cosine of the incident angle.

_[Source](https://blog.demofox.org/2016/09/21/path-tracing-getting-started-with-diffuse-and-emissive/)_

$$L_o( \omega_o)= L_e(\omega_o)+\int_{\Omega}{f(\omega_i, \omega_o)L_i(\omega_i)(\omega_i \cdot n)\mathrm{d}\omega_i}$$

- $L_o$ - light (radiance) directed outward along direction $\omega_o$ (light received by camera)
- $\omega_o$ - the direction of the outgoing light (view direction)
- $L_e$ - light (radiance) emitted
- $\int_{\Omega }\dots \,\mathrm{d}\omega_i$ - integral over $\Omega$
- $\Omega$ - the unit hemisphere centered around $n$ containing all possible values for $\omega_i$
- $f(\omega_i, \omega_o)$ - the [bidirectional reflectance distribution function (BRDF)](https://en.wikipedia.org/wiki/Bidirectional_reflectance_distribution_function), the proportion of light reflected from $\omega_i$ to $\omega_o$
- $\omega_i$ - the [negative](https://en.wikipedia.org/wiki/Bidirectional_reflectance_distribution_function#/media/File:BRDF_Diagram.svg) direction of the incoming light
- $L_i$ - light coming inward from direction $\omega_i$
- $n$ - the surface normal
- $\omega_i\cdot n$ - the weakening factor of outward irradiance due to incident angle $\cos \theta_{i}$ ([Lambert’s Cosine Law](https://en.wikipedia.org/wiki/Lambert%27s_cosine_law))

The above equation is a simplified version, and here is the a more extended explanation:

<img src="https://pbs.twimg.com/media/CHW_bGCUwAAIS1r.png" width="640" style="display:block; margin:auto;">
<div style="text-align:center">
<a href="https://twitter.com/levork/status/609603797258600448" style="color:lightgrey">click for source</a>
</div>
<br>

Here is another nice illustrated explanation:

<img src="https://i.redd.it/802mndge03t01.png" width="480" style="display:block; margin:auto;">
<div style="text-align:center">
<a href="https://www.reddit.com/r/visualizedmath/comments/8dofla/rendering_equation_explained/" style="color:lightgrey">click for source</a>
</div>
<br>
