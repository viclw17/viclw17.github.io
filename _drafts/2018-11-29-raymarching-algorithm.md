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

Fisrt of all, both raymarching and raytracing are algorithms for rendering 3D objects. No matter how, to render a certain 3D object we need to firstly define its shape.

In raytracing, the geometries are difined **explicitly** with vertices. Those vertices forms into triangles and then got connected edge by edge to form the final geometries - just like this kind of **low poly crafts**. Each vertex provides position information and each formed triangles provides suface normal etc.

![](https://pbs.twimg.com/media/Docd7meXoAA7b2J.jpg)

However in raymarching, 3D geometries are defined implicitly with mathmatical equations. For example, if plug a vertex position coordinates $(x, y, z)$ into this function

$$f(x, y, z) = \sqrt{x^2 + y^2 + z^2} - 1$$
