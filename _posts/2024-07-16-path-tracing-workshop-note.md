---
title: "Path Tracing Workshop Note"
layout: post
image: 2024-07-16-path-tracing-workshop-note\cover.png
---

Note for Intel Path-Tracing Workshop.

---

# Intel Path-Tracing Workshop
- [Path-Tracing Workshop Part 1: Write a **Ray Tracer**](https://www.intel.com/content/www/us/en/developer/videos/path-tracing-workshop-part-1.html#gs.c74n6i)
- [Path-Tracing Workshop Part 2: Write a **Path Tracer**](https://www.intel.com/content/www/us/en/developer/videos/path-tracing-workshop-part-2.html#gs.c74p8h)


This workshop shows how to:

- Implement ray tracing in software on the GPU.
- Use GLSL, Shadertoy, camera models, ray-triangle intersection tests, and ray-mesh intersection tests.
- Build on your ray tracer and implement a path tracer that renders a scene with full global illumination.
- Learn about fundamental concepts in physically based rendering such as global illumination, radiance, the rendering equation, Monte Carlo integration, and path tracing.
- Implement the Monte Carlo integration, and use it to compute direct illumination.
- Write your path tracer.

More about the workshop from the author can be found on his [blog](https://momentsingraphics.de/PathTracingWorkshop.html).

Course project result on shadertoy:

<iframe width="100%" height="360" frameborder="0" src="https://www.shadertoy.com/embed/Nlcczr?gui=true&t=10&paused=true&muted=true" allowfullscreen style="display:block; margin:auto;"></iframe>

# Ray Tracing
## A tiny ray tracer
[Back of the Business Card Ray Tracers](https://www.realtimerendering.com/blog/back-of-the-business-card-ray-tracers/)

<img src="https://www.realtimerendering.com/blog/wp-content/uploads/2021/10/image-2.png"  width="400" style="display:block; margin:auto;">

## Ray tracing vs Path tracing?
- **Ray tracing** is a **technique for modeling light transport** for use in a wide variety of rendering algorithms (esp. Path tracing) for generating digital images. *Foundation of path tracing.*
- **Path tracing** is a **Monte Carlo method** of rendering images of 3D scenes such that the **global illumination** is faithful to reality. *Path tracing is using ray tracing technique.*

> Because ray tracing is so incredibly simple, it should have been an obvious choice for implementing global illumination in computer graphics. -- [A Ray-Tracing Pioneer Explains How He Stumbled into Global Illumination](https://blogs.nvidia.com/blog/ray-tracing-global-illumination-turner-whitted/), a good read by J. Turner Whitted

More about ray tracing on Scratchapixel [Overview of the Ray-Tracing Rendering Technique](https://www.scratchapixel.com/lessons/3d-basic-rendering/ray-tracing-overview/ray-tracing-rendering-technique-overview.html)

## Workshop checklist
- Framework provided - shadertoy etc.
- Scene provided - triangles only cornell box
- Only diffuse surface
- no textures
-  no acceleration structures - bvh
- no clever sampling/ denoising
- running on GPUs because GLSL, so no graphics APIs

## GLSL refresher
### Features
- C-like shader language for **OpenGL, Vulkan, WebGL**
- For-loops of fixed max_length, no while-loops
- Fixed-size arrays, no pointers
- Indices must be constants

### Syntax
- Parameters in functions can be marked `in`, `out` or `inout`, copy values in, out or both
Built-in scalar, vector, matrix types: int float

- Built-in scalar, vector and matrix types:
    - `int float vec2 vec3 vec4 mat2 mat3`
- Constructor-like functions:
    - `vec3 v = vec3(x, y, z);` 
    - `mat3 A = mat3(col_0, col_1, col_2);`
- Standard functions/operators for geometric operations:
    - `dot (v, w)`
- Swizzles and array operators:
    - `vec2(v[ ], v[ ]) == v.yz` 
    - `v.xyz == v.rgb`
    - `float col_1_row_2 = A[1][2];`

## Scene representation
- Camera - generate camera ray, or `get_primary_ray_direction()`
- Triangle - 3D geometry with surface material info
- Mesh - Many triangles

### Camera
<img src="{{ site.url }}/images\2024-07-16-path-tracing-workshop-note\camera.png" width="400" style="display:block; margin:auto;">

### Shape - Triangle
Triangle definition. Here it contains both geo data and surface data, for simplicity.

In better implementation like in pbrt, geo data is the only focus for shape class, and a material class will focus on surface data including a reference to texture class. Above all, a primitive class will encompass both shape and material.

```
// A triangle along with some shading parameters
struct triangle_t {
    // The positions of the three vertices (v_0, v_1, v_2)
    vec3 positions[3];
    // A vector of length 1, orthogonal to the triangle (n)
    vec3 normal;
    // The albedo of the triangle (i.e. the fraction of
    // red/green/blue light that gets reflected) (a)
    vec3 color;
    // The radiance emitted by the triangle (for light sources) (L_e)
    vec3 emission;
};
```

## Ray tracing
### Intersection - Ray vs Triangle
<img src="{{ site.url }}/images\2024-07-16-path-tracing-workshop-note\triangle.png" width="400" style="display:block; margin:auto;">

Use **barycentric coordinate** to describe point on the triangle, and it is equal to the point on the ray, it indicates an intersection.

A mathmatic representation is as followed, and the goal here is to fomulate it toward getting the ray and 2 barycentric parameters.

<img src="{{ site.url }}/images\2024-07-16-path-tracing-workshop-note\triangle2.png" width="400" style="display:block; margin:auto;">

```
// Checks whether a ray intersects a triangle
// \param out_t The ray parameter at the intersection (if any) (t)
// \param origin The position at which the ray starts (o)
// \param direction The direction vector of the ray (d)
// \param tri The triangle for which to check an intersection
// \return true if there is an intersection, false otherwise
bool ray_triangle_intersection(out float out_t, vec3 origin, vec3 direction, triangle_t tri) {
    vec3 v0 = tri.positions[0];
    mat3 matrix = mat3(-direction, tri.positions[1] - v0, tri.positions[2] - v0);
    vec3 solution = inverse(matrix) * (origin - v0);
    out_t = solution.x;
    vec2 barys = solution.yz;
    return out_t >= 0.0 && barys.x >= 0.0 && barys.y >= 0.0 && barys.x + barys.y <= 1.0;
}
```

Note that the function outputs are in 2 ways:
- `bool` indicates the return value being a boolean, true if there is an intersection, false otherwise
- `out float out_t` parameter indicate it will capture the ray parameter at the intersection (if any) (t), aka the hitting point

### Intersection - Ray vs Mesh
```
// Checks whether a ray intersects any triangle of the mesh
// \param out_t The ray parameter at the closest intersection (if any) (t)
// \param out_tri The closest triangle that was intersected (if any)
// \param origin The position at which the ray starts (o)
// \param direction The direction vector of the ray (d)
// \return true if there is an intersection, false otherwise
bool ray_mesh_intersection(out float out_t, out triangle_t out_tri, vec3 origin, vec3 direction) {
    // Definition of the mesh geometry (exported from Blender)
    // ...

    // Find the nearest intersection across all triangles
    out_t = 1.0e38;
    for (int i = 0; i != TRIANGLE_COUNT; ++i) {
        float t;
        if (ray_triangle_intersection(t, origin, direction, tris[i]) && t < out_t) {
            out_t = t;
            out_tri = tris[i];
        }
    }
    return out_t < 1.0e38;
}
```

Note that the function outputs are in 3 ways:
- `bool` indicates the return value, true if there is an intersection, false otherwise
- `out triangle_t out_tri`: closest hit triangle
- `out float out_t`: ray parameter at the intersection (if smaller than max ray length)


<img src="{{ site.url }}/images\2024-07-16-path-tracing-workshop-note\triangle3.png" width="400" style="display:block; margin:auto;">


# Path tracing
## Global Illumination
Surfaces can be lit directly, but also indirectly, via paths of arbitrary length.
Path tracing starts at camera, finds a light when it is lucky.




