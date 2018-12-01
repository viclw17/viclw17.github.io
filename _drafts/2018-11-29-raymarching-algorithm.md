---
title: Raymarching Algorithm
date: 2018-11-29
tags:
- Algorithm
- Computer Graphics
---
Shadertoy is an amazing place to see all sorts of creative shader demos and get inspired. I notice most of the shaders there which depict certain 3D geometries - simple or extremely complex (could also be procedurally generated) - are drawn using raymarching algorithm.

# Raytracing & Raymarching
At first the algorithm sounds kind of magical, and the similarity of its name comparing with raytracing algorithm keeps me wondering. This time I want to dig deeper about it and get it documented for future reference.

---

Both raymarching and raytracing are **algorithms for rendering 3D objects**, and no matter how, to render a certain 3D object we need to firstly define its shape.

Raytracing is used mainly in PBR. In raytracing pipeline, geometries are usually prepared in **DCC(Digital Content Creation)** software and are defined **explicitly** with vertices. These vertices form into triangles and then got connected edge by edge to create the final geometries - just like this kind of **low poly crafts**.

<img src="https://pbs.twimg.com/media/Docd7meXoAA7b2J.jpg" width="320"  style="display:block; margin:auto;">

Each vertex provides position information and each formed triangle provides suface normal etc. Geometries are loaded into the pipeline specifically and ray-geometry intersection are calculated for the final rendering.

---

However when writing shader(GLSL), the geometries have to be defined within the shader. So a different approach is used. In raymarching pipeline, 3D geometries are defined **implicitly** with mathmatical equations.

For example, any 3D point that satisfies this equation is on the surface of a sphere with radius of 1 unit and origin at $(0, 0, 0)$.

$$f(x, y, z) = \sqrt{x^2 + y^2 + z^2} - 1$$
