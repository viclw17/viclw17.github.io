---
title: Raymarching Algorithm
date: 2018-11-29
tags:
- Algorithm
- Computer Graphics
---
<img src="{{ site.url }}/images/raymarching-0.png" width="640"  style="display:block; margin:auto;">
<!-- <figcaption style="text-align: center;">Continue on the previous post - more about my Siggraph 2018 journey. </figcaption> -->
<br>
[Shadertoy](https://www.shadertoy.com/view/4dKyRz) is an amazing place to see all sorts of creative shader demos and get inspired. I learned most of the shaders there which depict certain 3D geometries - simple or extremely complex (could also be procedurally generated) - are drawn using **raymarching algorithm**.

At first the algorithm sounds kind of magical, and the similarity of its name comparing with raytracing algorithm keeps me wondering. This time I want to dig deeper about it and get it documented for future reference.

# Geometry Construction
Both raymarching and raytracing are **algorithms for rendering 3D objects**, and no matter how, to render a certain 3D object we need to firstly construct/define its shape.

In raytracing pipeline, geometries are usually prepared in **DCC(Digital Content Creation)** software and are defined **explicitly** with vertices. These vertices form into triangles and then got connected edge by edge to create the final geometries - just like this kind of **low poly crafts**.

<img src="https://pbs.twimg.com/media/Docd7meXoAA7b2J.jpg" width="320"  style="display:block; margin:auto;">
</br>
<!-- Each vertex provides position information and each formed triangle provides surface normal etc. Geometries are loaded into the pipeline specifically and ray-geometry intersection are calculated for the final rendering. -->

However when writing shader with GLSL, the geometries have to be defined within the shader. So a different approach is used. In raymarching pipeline, 3D geometries are defined **implicitly** with **mathematical equations**.

## SDF
For example, any 3D point that satisfies this equation is on the surface of a sphere with radius of 1 unit and origin at $(0, 0, 0)$:

$$f(x, y, z) = \sqrt{x^2 + y^2 + z^2} - 1$$

- $f(x, y, z) < 0$, the point is inside the sphere;
- $f(x, y, z) > 0$, the point is outside the sphere;
- $f(x, y, z) = 0$, the point is on the sphere surface.

because the result $f(x, y, z)$ is also the **distance** between the point and the sphere surface, and the **sign** of it tells if the point is inside/outside/on the sphere surface, so this function is also called **Signed Distance Function (SDF)**.

### Sphere
Code for previous sphere example.
```c
// for sphere with radius r
float sphereSDF(vec3 p, float r) {
    return length(p) - r;
}
```

### Box/Cube
```c
// for box with extends b (length, width, height)
// vec3 d records the distance between p and box surface on 3 axis
float boxSDF(vec3 p, vec3 b)
{
      d = abs(p) - b;
    return length(max(d,0)) + min(max(d.x,max(d.y,d.z)),0);
}
```  
If ```d.x < 0```, then ```-d.x < p.x < d.x``` which means p has coordinate that smaller than the box extend on X axis. Same explanation for y and z coordinates. So if ```vec3 d``` has all xyz coordinates less than 0, then p is inside the box.

> Reference: [GLSL BUILT-IN FUNCTIONS](http://www.shaderific.com/glsl-functions/).
The max function returns the larger of the two arguments. The input parameters can be floating scalars or float vectors. In case of float vectors the operation is done **component-wise**.

For the first part of the return, ```max(d,0)``` returns coordinates of d and they will be either bigger than or equal to 0 - which tells that p is outside or on the surface of the box. Then ```length()``` calculates how far it is to the surface.

For the second part of the return, ```min(max(d.x,max(d.y,d.z)),0)``` compares the largest coordinate of d with 0 and return the smaller one. The result will be either smaller than or equal to 0 - which tells that p is inside or on the surface of the box.

Final result combines both considerations.

```c
// for unit cube, with better annotation
float cubeSDF(vec3 p) {
    // If d.x < 0, then -1 < p.x < 1, and same logic applies to p.y, p.z
    // So if all components of d are negative, then p is inside the unit cube
    vec3 d = abs(p) - vec3(1);

    // Assuming p is inside the cube, how far is it from the surface?
    // Result will be negative or zero.
    float insideDistance = min(max(d.x, max(d.y, d.z)), 0);

    // Assuming p is outside the cube, how far is it from the surface?
    // Result will be positive or zero.
    float outsideDistance = length(max(d, 0));

    return insideDistance + outsideDistance;
}
```
### More SDF examples
Inigo Quilez's blog post - [distance functions](http://iquilezles.org/www/articles/distfunctions/distfunctions.htm)

<iframe width="100%" height="360" frameborder="0" src="https://www.shadertoy.com/embed/Xds3zN?gui=true&t=10&paused=true&muted=false" allowfullscreen style="display:block; margin:auto;"></iframe>
<br>

# Raymarching
Now we can describe 3D objects using SDF which returns signed distance between any point and 3D surface. To render them we will be using raymarching algorithm. From here I'm mainly citing Jamie Wong’s amazing blog post - [Ray Marching and Signed Distance Functions](http://jamie-wong.com/2016/07/15/ray-marching-signed-distance-functions/).

>Just as in raytracing, we select a position for the camera, put a grid in front of it, send rays from the camera through each point in the grid, with each grid point corresponding to a pixel in the output image.

<br>
<img src="https://upload.wikimedia.org/wikipedia/commons/8/83/Ray_trace_diagram.svg" width="640"  style="display:block; margin:auto;">
<!-- <figcaption style="text-align: center;">Continue on the previous post - more about my Siggraph 2018 journey. </figcaption> -->
<br>

>In raymarching, to find the intersection between the view ray and the scene, we start at the camera, and move a point along the view ray, bit by bit. At each step, we ask “Is this point inside the scene surface?”, or alternately phrased, “Does the SDF evaluate to a negative number at this point?“. If it does, we’re done! We hit something. If it’s not, we keep going up to some maximum number of steps along the ray.

>We could just step along a very small increment of the view ray every time, but we can do much better than this (both in terms of speed and in terms of accuracy) using “**sphere tracing**”.

</br>
<iframe width="100%" height="360" frameborder="0" src="https://www.shadertoy.com/embed/4dKyRz?gui=true&t=10&paused=true&muted=false" allowfullscreen style="display:block; margin:auto;"></iframe>
</br>

```c
/**
 * Return the shortest distance from the eyepoint to the scene surface along
 * the marching direction. If no part of the surface is found between start and end,
 * return end.
 *
 * eye: the eye point, acting as the origin of the ray
 * marchingDirection: the normalized direction to march in
 * start: the starting distance away from the eye
 * end: the max distance away from the ey to march before giving up
 */
 float shortestDistanceToSurface(vec3 eye, vec3 marchingDirection, float start, float end) {
     float depth = start;
     for (int i = 0; i < MAX_MARCHING_STEPS; i++) {
         float dist = sceneSDF(eye + depth * marchingDirection);
         if (dist < EPSILON) {
             return depth;
         }
         depth += dist;
         if (depth >= end) {
             return end;
         }
     }
     return end;
 }
```
To get the ```marchingDirection```,

```c
/**
 * Return the normalized direction to march in from the eye point for a single pixel.
 *
 * fieldOfView: vertical field of view, in degrees
 * size: resolution of the output image
 * fragCoord: the x,y coordinate of the pixel in the output image
 */
vec3 rayDirection(float fieldOfView, vec2 size, vec2 fragCoord) {
    vec2 xy = fragCoord - size / 2.0; // translate screenspace to centre
    float z = size.y / tan(radians(fieldOfView) / 2.0);
    return normalize(vec3(xy, -z));
}
```
To finally render:
```c
void mainImage( out vec4 fragColor, in vec2 fragCoord )
{
	vec3 dir = rayDirection(45.0, iResolution.xy, fragCoord);
    vec3 eye = vec3(0.0, 0.0, 5.0);
    float dist = shortestDistanceToSurface(eye, dir, MIN_DIST, MAX_DIST);

    // Didn't hit anything
    if (dist > MAX_DIST - EPSILON) {
        fragColor = vec4(0.0, 0.0, 0.0, 0.0);
		return;
    }

    // Hit on the surface
    fragColor = vec4(1.0, 0.0, 0.0, 1.0);
}
```

<!-- # Surface Normals and Lighting -->

TBC
