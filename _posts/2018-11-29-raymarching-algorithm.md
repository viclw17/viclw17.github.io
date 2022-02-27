---
title: Raymarching Algorithm
layout: post
image: 2018-11-29-raymarching-algorithm/raymarching-0.png
---

<img src="{{ site.url }}/images/2018-11-29-raymarching-algorithm/raymarching-0.png" width="640"  style="display:block; margin:auto;">
<!-- <figcaption style="text-align: center;">Continue on the previous post - more about my Siggraph 2018 journey. </figcaption> -->
<br>

[Shadertoy](https://www.shadertoy.com/view/4dKyRz) is an amazing place to see all sorts of creative shader demos and get inspired. I learned most of the shaders there which depict certain 3D geometries - simple or extremely complex (could also be procedurally generated) - are drawn using **raymarching algorithm**.

At first the algorithm sounds kind of magical, and the similarity of its name comparing with raytracing algorithm keeps me wondering. This time I want to dig deeper about it and get it documented for future reference.

---

# Geometry Construction
Both raymarching and raytracing are **algorithms for rendering 3D objects**, and no matter how, to render a certain 3D object we need to firstly construct/define its shape.

## Explicitly
Generally speaking an explicit geometry is defined with a range of parameterization functions.

For example, for a sphere with center at $(x_0,y_0,z_0)$ and radius $r$:

$${\begin{aligned}f(x)&=x_{0}+r\sin \varphi \;\cos \theta \\f(y)&=y_{0}+r\sin \varphi \;\sin \theta \qquad (0\leq \varphi \leq \pi ,\;0\leq \theta <2\pi )\\f(z)&=z_{0}+r\cos \varphi \,\end{aligned}}$$

However, in raytracing pipeline, geometries are usually <!-- prepared in **DCC(Digital Content Creation)** software and are--> defined **explicitly** with **vertices**. These vertices form into triangles and then got connected edge by edge to create the final geometries - just like this kind of **low poly crafts**. This is called geometry **polygonization**.

<!-- Each vertex provides position information and each formed triangle provides surface normal etc. Geometries are loaded into the pipeline specifically and ray-geometry intersection are calculated for the final rendering.

However when writing shader with GLSL, the geometries have to be defined within the shader. So a different approach is used. In raymarching pipeline, 3D geometries are defined **implicitly** with **mathematical equations**.-->

## Implicitly - SDF
A different approach is to define 3D geometries **implicitly** with mathematical equations.

For example, any 3D point that satisfies this equation is on the surface of a sphere with radius of 1 unit and origin at $(0, 0, 0)$:

$$f(x, y, z) = \sqrt{x^2 + y^2 + z^2} - 1$$

- $f(x, y, z) < 0$, the point is inside the sphere;
- $f(x, y, z) > 0$, the point is outside the sphere;
- $f(x, y, z) = 0$, the point is on the sphere surface.

because the result $f(x, y, z)$ is also the **distance** between the point and the sphere surface, and the **sign** of it tells if the point is inside/outside/on the sphere surface, so this function is also called **Signed Distance Function (SDF)**.

### Sphere
Code for previous sphere SDF example.
```c
// for sphere with radius r
float sphereSDF(vec3 p, float r) {
    return length(p) - r;
}
// for unit sphere with radius r = 1
float sphereSDF(vec3 p) {
    return length(p) - 1;
}
```
>Note: The ```length``` function returns the length of a vector defined by the Euclidean norm, i.e. the square root of the sum of the squared components. The input parameter can be a **float scalar** or a **float vector**. In case of a floating scalar the ```length``` function is trivial and returns the **absolute value**.

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

> Note: The ```max``` function returns the larger of the two arguments. The input parameters can be floating scalars or float vectors. In case of float vectors the operation is done **component-wise**.

For the first part of the return, ```max(d,0)``` returns coordinates of d and they will be either bigger than or equal to 0 - which tells that p is outside or on the surface of the box. Then ```length()``` calculates how far it is to the surface.

For the second part of the return, ```min(max(d.x,max(d.y,d.z)),0)``` compares the largest coordinate of d with 0 and return the smaller one. The result will be either smaller than or equal to 0 - which tells that p is inside or on the surface of the box.

Final result combines both considerations:
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

# Raymarching Algorithm
Now we can describe 3D objects using SDF which returns signed distance between any point and 3D surface. To render them we will be using raymarching algorithm. From here I'm mainly citing Jamie Wong’s amazing blog post - [Ray Marching and Signed Distance Functions](http://jamie-wong.com/2016/07/15/ray-marching-signed-distance-functions/).

>Just as in raytracing, we select a position for the camera, put a grid in front of it, send rays from the camera through each point in the grid, with each grid point corresponding to a pixel in the output image.

<br>
<img src="https://upload.wikimedia.org/wikipedia/commons/8/83/Ray_trace_diagram.svg" width="480"  style="display:block; margin:auto;">
<br>
<figcaption style="text-align: center;">Image from - <a href="https://en.wikipedia.org/wiki/Ray_tracing_(graphics)">Wikipedia</a></figcaption>
<br>

>In raymarching, to find the intersection between the view ray and the scene, we start at the camera, and move a point along the view ray, bit by bit. At each step, we ask “Is this point inside the scene surface?”, or alternately phrased, **"Does the SDF evaluate to a negative number at this point?"**. If it does, we’re done! We hit something. If it’s not, we keep going up to some maximum number of steps along the ray.

>We could just step along a very small increment of the view ray every time, but we can do much better than this (both in terms of speed and in terms of accuracy) using “**sphere tracing**”.

<iframe width="640" height="360" frameborder="0" src="https://www.shadertoy.com/embed/4dKyRz?gui=true&t=10&paused=true&muted=false" allowfullscreen style="display:block; margin:auto;"></iframe>
<br>
<figcaption style="text-align: center;"><a href="https://www.shadertoy.com/view/4dKyRz">Ray Marching Demo For Beginner</a> by Trashe725</figcaption>
<br>

GLSL code implementation for raymartching algorithm:
- ```eye```: the eye/camera position, acting as the ray origin
- ```marchingDirection```: the normalized ray direction to march in
- ```start```: the starting distance away from the eye/camera
- ```end```: the max distance away from the eye/camera to march before giving up

```c
/**
 * Return the shortest distance from the eyepoint to the scene surface along
 * the marching direction. If no part of the surface is found between start and end,
 * return end.
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
## Amazing interactive tutorial
<iframe width="640" height="360" frameborder="0" src="https://www.shadertoy.com/embed/4dSfRc?gui=true&t=10&paused=true&muted=false" allowfullscreen style="display:block; margin:auto;"></iframe>
<br>
<figcaption style="text-align: center;"><a href="https://www.shadertoy.com/view/4dSfRc">[SH17C] Raymarching tutorial</a> by reinder</figcaption>

# Rendering
<!-- To finally render, we need to set up camera model and cast rays. Conventionally, we put camera at origin (0,0,0) in the world space pointing to the negative Z axis. Now we need ```fieldOfView``` and ```size``` to define the camera model where:
- ```fieldOfView```: vertical field of view, in degrees
- ```size```: resolution of the output image, in ```vec2```

We cast rays from camera position to every single pixel gird. ```fragCoord``` is the coordinate of the pixel in the output image, which works as the x,y coordinates of the points where rays hit on the image plane. But how to get the z coordinate for the hit points? This can be derived from ```fieldOfView``` and ```size```.

<img src="{{ site.url }}/images/2018-11-29-raymarching-algorithm/camera-model-summary-0.jpg" width="640"  style="display:block; margin:auto;">

```c
/**
 * Return the normalized direction to march in from the eye point for a single pixel.
 */
vec3 rayDirection(float fieldOfView, vec2 size, vec2 fragCoord) {
    vec2 xy = fragCoord - size / 2.0; // translate screenspace to centre
    float theta = radians(fieldOfView) / 2.0;
    float halfHeight = size.y / 2.0;
    float z = halfHeight / tan(theta);
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
 -->

```c
const int MAX_MARCHING_STEPS = 255;
const float MIN_DIST = 0.0;
const float MAX_DIST = 100.0;
float EPSILON = 0.0001;

struct Ray{
	vec3 origin;
    vec3 direction;
};

float SphereSDF(vec3 samplePoint) {
    return length(samplePoint) - 1.;
}

float ShortestDistanceToSurface(Ray ray, float start, float end) {
    float depth = start;
    for (int i = 0; i < MAX_MARCHING_STEPS; i++) {
        float dist = SphereSDF(ray.origin + depth * ray.direction);
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

void mainImage( out vec4 fragColor, in vec2 fragCoord )
{
    // Scale and bias uv
    // [0.0, iResolution.x] -> [0.0, 1.0]
    // [0.0, 1.0] 			-> [-1.0, 1.0]
    vec2 xy = fragCoord / iResolution.xy;
	xy = xy * 2.- vec2(1.);
	xy.x *= iResolution.x/iResolution.y;

    // SphereSDF position at (0,0,0)

    vec3 pixelPos = vec3(xy, 2.); // Image plane at (0,0,2)
    vec3 eyePos = vec3(0.,0.,5.); // Camera position at (0,0,5)
    vec3 rayDir = normalize(pixelPos - eyePos);

    float dist = shortestDistanceToSurface(eyePos, rayDir, MIN_DIST, MAX_DIST);

    // Didn't hit anything
    if (dist > MAX_DIST - EPSILON) {
        fragColor = vec4(0.0, 0.0, 0.0, 0.0);
		return;
    }

    // Hit on the surface
    fragColor = vec4(1.0);
}
```

<iframe width="640" height="360" frameborder="0" src="https://www.shadertoy.com/embed/XtGfWG?gui=true&t=10&paused=true&muted=false" allowfullscreen style="display:block; margin:auto;"></iframe>
<br>

# Raymartching vs Raytracing
The main difference is the function ```float RaySphereIntersection(Ray ray, Sphere sphere)```. Explanation see previous post --> [Raytracing - Ray Sphere Intersection](http://viclw17.github.io/2018/07/16/raytracing-ray-sphere-intersection/)
```c
struct Ray{
	vec3 origin;
    vec3 direction;
};

struct Sphere{
	vec3 center;
    float radius;
};

float RaySphereIntersection(Ray ray, Sphere sphere){
    vec3 oc = ray.origin - sphere.center;
    float a = dot(ray.direction, ray.direction);
    float b = 2.*dot(oc,ray.direction);
    float c = dot(oc,oc)-sphere.radius*sphere.radius;
    float discriminant = b*b-4.*a*c;

    if(discriminant<0.)
        return -1.;
    else
        return (-b-sqrt(discriminant))/(2.*a);
}

void mainImage( out vec4 fragColor, in vec2 fragCoord )
{
    // Scale and bias uv
    // [0.0, iResolution.x] -> [0.0, 1.0]
    // [0.0, 1.0] 			-> [-1.0, 1.0]
    vec2 xy = fragCoord / iResolution.xy;
	xy = xy * 2.- vec2(1.);
	xy.x *= iResolution.x/iResolution.y;

    Sphere sphere = Sphere(vec3(0.), 1.0); // Sphere position at (0,0,0)

	vec3 pixelPos = vec3(xy, 2.); // Image plane at (0,0,2)
    vec3 eyePos = vec3(0.,0.,5.); // Camera position at (0,0,5)
    vec3 rayDir = normalize(pixelPos - eyePos);

    float dist = RaySphereIntersection(Ray(eyePos, rayDir), sphere);

    // Didn't hit anything
    if (dist < 0.) {
        fragColor = vec4(0.);
		return;
    }

    // Hit on the surface
    fragColor = vec4(1.,0.,0.,1.);
}
```

<iframe width="640" height="360" frameborder="0" src="https://www.shadertoy.com/embed/wdsGR7?gui=true&t=10&paused=true&muted=false" allowfullscreen style="display:block; margin:auto;"></iframe>
