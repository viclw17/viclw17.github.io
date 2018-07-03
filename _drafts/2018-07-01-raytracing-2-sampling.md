---
title: Raytracer Case Study - Part 2 - Sampling
date: 2018-07-01
tags:
- Computer Graphics
- Raytracing
- PBR
---
# Sampling
## Uniform and Non-uniform Sampling
There is two kind of sampling, uniform and non-uniform. Uniform sampling guarantee that all the coordinates have the same probability to be chosen.
![](https://blog.thomaspoulet.fr/assets/content/Sampling/fig1.png)
- In perfect mirror conditions incident rays are generated using the law of reflection.
- In glossy materials, incident rays are generated on the unit Hemisphere with the reflection direction.
- In pure diffuse conditions, incident rays are sampled uniformly on the unit sphere.
- Pure diffuse surfaces are only theoretical, but they makes a good approximations of what we can find in the real world.
## Hemisphere: Spherical Coordinates
The Cartesian coordinates may be retrieved from the spherical coordinates (radius $r$, inclination $θ$, azimuth $φ$), where $r ∈ [0, ∞)$, $θ ∈ [0, π]$, $φ ∈ [0, 2π)$, by:

$$\begin{aligned}x&=r\,\sin \theta \,\cos \varphi \\y&=r\,\sin \theta \,\sin \varphi \\z&=r\,\cos \theta \end{aligned}$$

<img src="https://upload.wikimedia.org/wikipedia/commons/4/4f/3D_Spherical.svg" width="240"  style="display:block; margin:auto;">
<br>

---

# Unit Disk
The (open) unit disk can also be considered to be the region in the complex plane defined by {z:|z|<1}, where |z| denotes the complex modulus. (The closed unit disk is similarly defined as {z:|z|<=1}.

# Disk Point Picking
http://mathworld.wolfram.com/DiskPointPicking.html
<img src="http://mathworld.wolfram.com/images/eps-gif/CircularDistribution_1000.gif" width="320"  style="display:block; margin:auto;">

To generate random points over the unit disk, it is incorrect to use two uniformly distributed variables r in [0,1] and theta in [0,2pi) and then take

x	=	rcostheta
y	=	rsintheta.

Because the area element is given by dA=2pirdr, 	

this gives a concentration of points in the center (left figure above).

The correct transformation is instead given by

x	=	sqrt(r)costheta
y	=	sqrt(r)sintheta
(right figure above).

The probability function for distance d from the center of a point picked at random in a unit disk is

 P(d)=2d. 	
The raw moments are therefore given by

 mu_n^'=2/(2+n), 	

giving a mean distance of d^_=2/3.
