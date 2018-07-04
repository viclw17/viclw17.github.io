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
[Smallpt](http://www.kevinbeason.com/smallpt/) is a global illumination renderer. It is 99 lines of C++ and open source. It is quite an amazing raytracing practice.

## Features
- Global illumination via unbiased Monte Carlo path tracing
- 99 lines of 72-column (or less) open source C++ code
- Multi-threading using OpenMP
- Soft shadows from diffuse luminaire
- Specular, Diffuse, and Glass BRDFs
- Antialiasing via super-sampling with importance-sampled tent distribution, and 2x2 subpixels
- Ray-sphere intersection
- Modified Cornell box scene description
- Cosine importance sampling of the hemisphere for diffuse reflection
- Russian roulette for path termination
- Russian roulette and splitting for selecting reflection and/or refraction for glass BRDF
- With minor changes compiles to a 4 KB binary (less than 4096 bytes)

# System Breakdown
Source is a [GLSL shader implementation](https://www.shadertoy.com/view/4sfGDB) of [Smallpt](http://www.kevinbeason.com/smallpt/) .

## Preliminaries
```c
#define SAMPLES 16
#define MAXDEPTH 4
// Not used for now
//#define DEPTH_RUSSIAN 2
#define PI 3.14159265359
#define DIFF 0 // Diffuse reflection
#define SPEC 1 // Specular reflection
#define REFR 2 // Refraction
#define NUM_SPHERES 9
// random number generator
float rand() {
  return fract(sin(seed++)*43758.5453123);
}
```
## Geometries
```c
// origin, direction
// vec3,   vec3
struct Ray {
    vec3 o, d;     // origin, direction
};

// radius, position, emissive, color，reflection
// float,  vec3,     vec3,     vec3,  int
struct Sphere {
    float r;      // radius
    vec3 p, e, c; // position, emissive, color
    int refl;     // reflection/refraction
};

// hard coded scene description (Cornell's Box)
Sphere spheres[NUM_SPHERES];
void initSpheres() {
    float scale = 0.;
    // walls
    spheres[0] = Sphere(1e5, vec3(-1e5+01., 40.8, 81.6),vec3(0.), REDCOLOR,  DIFF); // left
    spheres[1] = Sphere(1e5, vec3( 1e5+99., 40.8, 81.6),vec3(0.), BLUECOLOR, DIFF); // right
    spheres[2] = Sphere(1e5, vec3(50., 40.8, -1e5),	vec3(0.), GRAYCOLOR, DIFF); // back
    spheres[3] = Sphere(1e5, vec3(50., 40.8,  1e5+170.),vec3(0.), GRAYCOLOR, DIFF); // front
    spheres[4] = Sphere(1e5, vec3(50., -1e5, 81.6),	vec3(0.), GRAYCOLOR, DIFF); // floor
    spheres[5] = Sphere(1e5, vec3(50.,  1e5+81.6, 81.6),vec3(0.), GRAYCOLOR, DIFF); // ceiling
    // spheres
    spheres[6] = Sphere(16.5, vec3(27., 16.5, 47.), vec3(0.), WHITECOLOR, SPEC);
    spheres[7] = Sphere(16.5, vec3(73., 16.5, 78.), vec3(0.), WHITECOLOR, DIFF);
    // lighting sphere
    spheres[8] = Sphere(600., vec3(50., 681.33, 81.6), vec3(10.), WHITECOLOR, DIFF);
}

```
## Sphere Intersection
## Scene Intersection
```c
// Ray-scene intersection
int scene_intersect(Ray r, out float t, out Sphere s, int avoid) {
    int id = -1;
    t = 1e5;
    s = spheres[0];
    for (int i = 0; i < NUM_SPHERES; ++i) {
    	Sphere S = spheres[i];
    	float d = sphere_intersect(S, r);
    	if (i!=avoid && d!=0. && d<t) { t = d; id = i; s=S; }
    }
    return id;
}
```

## Radiance Evaluation

## Main Function
```c
// main function, runs on every fragment
void main(void) {
    initSpheres(); // initialize scene
    seed = resolution.y * gl_FragCoord.x / resolution.x + gl_FragCoord.y / resolution.y;
    vec2 uv = 2. * gl_FragCoord.xy / resolution.xy - 1.; // normalize uv coordinates
    // set up camera coordinates
    vec3 camPos = vec3(50,40,160);
    vec3 cz = normalize(vec3(50., 40., 81.6) - camPos);
    vec3 cx = vec3(1., 0., 0.);
    vec3 cy = normalize(cross(cx, cz)); cx = cross(cz, cy);
    vec3 color = vec3(0.);
    Ray ray; // create ray from every fragment
    for (int i = 0; i < SAMPLES; ++i)
    {   
        // compute ray direction using cam.d, cx, cy
        ray = Ray(camPos, normalize(.53135 * (resolution.x/resolution.y*uv.x * cx + uv.y * cy) + cz));
        // Use radiance function to estimate radiance
    	color += radiance(ray);
    }
    // output color
    gl_FragColor = vec4(pow(clamp(color/float(SAMPLES), 0., 1.), vec3(1./2.2)), 1.);
}
```
