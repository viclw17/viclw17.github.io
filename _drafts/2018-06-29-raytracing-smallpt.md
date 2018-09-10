---
title: Raytracer Case Study - Smallpt Breakdown
date: 2018-06-29
tags:
- Computer Graphics
- Raytracing
- PBR
---
<img src="http://www.kevinbeason.com/smallpt/result_25k.png" width="480"  style="display:block; margin:auto;">
In the following few posts I want to dive deep into a [GLSL shader implementation](https://www.shadertoy.com/view/4sfGDB) of [Smallpt](http://www.kevinbeason.com/smallpt/) raytracer and go over all the related knowledge.

# Smallpt
[Smallpt](http://www.kevinbeason.com/smallpt/) is a global illumination renderer. It is written by _Kevin Beason_ and is quite an amazing raytracing implementation.

## Features
- Global illumination via unbiased Monte Carlo path tracing
- Multi-threading using OpenMP
- Soft shadows from diffuse luminaire
- Specular, Diffuse, and Glass BRDFs
- Antialiasing via super-sampling with importance-sampled tent distribution, and 2x2 subpixels
- Ray-sphere intersection
- Modified Cornell box scene description
- Cosine importance sampling of the hemisphere for diffuse reflection
- Russian roulette for path termination
- Russian roulette and splitting for selecting reflection and/or refraction for glass BRDF

## Code

# System Breakdown
Source is a [GLSL shader implementation](https://www.shadertoy.com/view/4sfGDB) of [Smallpt](http://www.kevinbeason.com/smallpt/) .
