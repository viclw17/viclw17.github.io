---
title: "Path Tracing Workshop Note"
layout: post
image: 2024-07-15-path-tracing-workshop-note\cover.png
---

<img src="{{ site.url }}/images/2024-07-15-path-tracing-workshop-note\cover.png" style="display:block; margin:auto;">

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
<img src="{{ site.url }}/images\2024-07-15-path-tracing-workshop-note\camera.png" width="400" style="display:block; margin:auto;">

### Shape - Triangle
Triangle definition. Here it contains both geo data and surface data, for simplicity.

In better implementation like in pbrt, geo data is the only focus for shape class, and a material class will focus on surface data including a reference to texture class. Above all, a primitive class will encompass both shape and material.

```glsl
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
<img src="{{ site.url }}/images\2024-07-15-path-tracing-workshop-note\triangle.png" style="display:block; margin:auto;">

Use **barycentric coordinate** to describe point on the triangle, and it is equal to the point on the ray, it indicates an intersection.

A mathmatic representation is as followed, and the goal here is to fomulate it toward getting the ray and 2 barycentric parameters.

<img src="{{ site.url }}/images\2024-07-15-path-tracing-workshop-note\triangle2.png" style="display:block; margin:auto;">

```glsl
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
<img src="{{ site.url }}/images\2024-07-15-path-tracing-workshop-note\triangle3.png" style="display:block; margin:auto;">

```glsl
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

# Path tracing

## Global Illumination
Surfaces can be lit directly, but also indirectly, via paths of arbitrary length.
Path tracing starts at camera, finds a light when it is lucky.

## Radiance
$L_(x,w)$ is basically color for ray $x+tw$. It is a **plenoptic function / radiance field**.

> The plenoptic illumination function is an idealized function used in computer graphics to express the image of a scene from **any possible viewing position** at **any viewing angle** at **any pointin time**.

<img src="https://upload.wikimedia.org/wikipedia/commons/d/d0/Plenoptic-function-a.png" width = "300" style="display:block; margin:auto;">


So the final pixel color is the radiance for camera ray.

Radiance is constant along rays in vacuum.
<!-- , hence $$L(y,w)=L(x.w)$$ -->

Ray tracing is transporting radiance, describes how light propogates in empty space.

### More from pbrt
Radiance measures Irradiance with respect to solid angles. Definition:

$$L = \frac{dE_{w}}{dw}$$

where, $E_w$ is the irradiance at the surface that is perpendicular to the direction $w$: $E_{\omega} =  \frac{d\phi}{dA^ \bot}$, so

$$L = \frac{dE_w}{dw} = \frac{d\phi}{d\omega \cdot dA^ \bot}$$

which means, radiance is the flux density per unit area, per unit solid angle.

> It is the limit of: the measurement of incident light at the surface, as a cone of incident directions of interest dw becomes very small, and as the local area of interest on the surface dA also becomes very small.



## Irradiance
We don't just have to deal with empty space, we also have to figure out how light **interact with the surface** - defined by irradiance.

Irradiance is the weighted integral over radiance:

$$E(x,n(x)) = \int_{\Omega(x)}(L(x,w) n(x) \cdot w dw)$$

### More from pbrt
From a differential perspective, irradiance is the average density of power over the area. Taking the limit of differential power per differential area at a point p, we got:

$$E(p) = \frac{d\phi(p)}{dA}$$

It is guided by **Lambert’s Law**.

## Rendering equation
**Note that this is the simplified version for the workshop:*
$$L_o(x) = L{e}(x) + \frac{a(x)}{\pi} \int_{\Omega(x)}(L(x,w) n(x) \cdot w dw)$$

- result: outgoing radiance $L_o(x)$ for diffuse surface at x
- compute incoming irradiance $E(x,n(x))$, = total light reaching x
- multiply by surface color $a(x)$
- divide by $\pi$ to ensure energy conservation
- add light emitted at x, 0 if x is not a light source

We have to integrate over $\Omega(x)$, which contains infinite many of incoming direction vectors. We need $L(x,w)$ which equal to $L{o}(y)$ where $y = ray\_intersection(x,w) = x + tw$, and accordingly there are infinite many of point $y$.

## Monte Carlo Integration
Instead, pick(sample) $w1$ at random, we have:

$$\int_{\Omega(x)}(L(x,w) n(x) \cdot w dw) \approx 2{\pi}L(x,w_1) n(x) \cdot w_1$$

when sample for many (towards infinite) times:

$$\int_{\Omega(x)}(L(x,w) n(x) \cdot w dw) \approx 2{\pi} \frac{1}{N}\underset{j = 1}{\overset{N}{\sum }} L(x,w_j) n(x) \cdot w_j$$

### Uniform hemisphere sampling
<iframe width="100%" height="360" frameborder="0" src="https://www.shadertoy.com/embed/7t3yRn?gui=true&t=10&paused=true&muted=true" allowfullscreen style="display:block; margin:auto;"></iframe>

Code:

```glsl
// Given uniform random numbers u_0, u_1 in [0,1)^2, this function returns a
// uniformly distributed point on the unit sphere (i.e. a random direction)
// (omega)
vec3 sample_sphere(vec2 random_numbers) {
    float z = 2.0 * random_numbers[1] - 1.0;
    float phi = 2.0 * M_PI * random_numbers[0];
    float x = cos(phi) * sqrt(1.0 - z * z);
    float y = sin(phi) * sqrt(1.0 - z * z);
    return vec3(x, y, z);
}


// Like sample_sphere() but only samples the hemisphere where the dot product
// with the given normal (n) is >= 0
vec3 sample_hemisphere(vec2 random_numbers, vec3 normal) {
    vec3 direction = sample_sphere(random_numbers);
    if (dot(normal, direction) < 0.0)
        direction -= 2.0 * dot(normal, direction) * normal;
    return direction;
}
```

### Psendorandom number generator
```glsl
// A pseudo-random number generator
// \param seed Numbers that are different for each invocation. Gets updated so
//             that it can be reused.
// \return Two independent, uniform, pseudo-random numbers in [0,1) (u_0, u_1)
vec2 get_random_numbers(inout uvec2 seed) {
    // This is PCG2D: https://jcgt.org/published/0009/03/02/
    seed = 1664525u * seed + 1013904223u;
    seed.x += 1664525u * seed.y;
    seed.y += 1664525u * seed.x;
    seed ^= (seed >> 16u);
    seed.x += 1664525u * seed.y;
    seed.y += 1664525u * seed.x;
    seed ^= (seed >> 16u);
    // Convert to float. The constant here is 2^-32.
    return vec2(seed) * 2.32830643654e-10;
}

// ...
// Use a different seed for each pixel and each frame
uvec2 seed = uvec2(pixel_coord) ^ uvec2(iFrame << 16, iFrame << 16 + 237);

// This gives 2 uniform random numbers in [0,1)
vec2 rands_0 = get_random_numbers(seed);
// These are different random numbers because seed has changed
vec2 rands_1 = get_random_numbers(seed);
```

## Direct Illumination
$$L(x) = L_e + \frac{a(x)}{\pi} L_e(y) n(x) \cdot w$$
$$y = ray\_intersection(x,w) = x + tw$$

## Direct + Indirect Illumination
Camera ray $x_0, w_0$, want to estimate $L(x_0, w_0)$, $x_1 = ray\_intersection(x_0,w_0)$, setting Monte Carlo $N = 1$:

$$L(x_0, w_0) = L_o(x_1) \approx L_e(x_1) + \frac{a(x_1)}{\pi} 2{\pi} L(x_1,w_1)n(x_1) \cdot w_1$$

then from point $x_1$, ray trace to another random direction for next intersection point, and we will have:

$$L(x_1, w_1) = L_o(x_2) \approx L_e(x_2) + \frac{a(x_2)}{\pi} 2{\pi} L(x_2,w_1)n(x_2) \cdot w_2$$

and this keeps repeating.

## Flatten recursion into loop
<img src="{{ site.url }}/images/2024-07-15-path-tracing-workshop-note\pt1.png" style="display:block; margin:auto;">

Add emission and update throughput weight $T_j$ in each iteration:

<img src="{{ site.url }}/images/2024-07-15-path-tracing-workshop-note\pt2.png" style="display:block; margin:auto;">

```glsl
// Performs path tracing: It starts with the given ray. If this ray intersects
// a triangle, a new random ray is traced iteratively, up to a fixed limit.
// \param origin The position at which the ray starts (x_j)
// \param direction The direction vector of the ray (omega_j)
// \param seed Needed for get_random_numbers()
// \return A noisy estimate of the reflected and emitted radiance at the point
//         intersected by the ray (i.e. the color) (L_o(x))
vec3 get_ray_radiance(vec3 origin, vec3 direction, inout uvec2 seed) {
    vec3 radiance = vec3(0.0);
    vec3 throughput_weight = vec3(1.0);
    for (int i = 0; i != MAX_PATH_LENGTH; ++i) {
        float t;
        triangle_t tri;
        if (ray_mesh_intersection(t, tri, origin, direction)) {
            radiance += throughput_weight * tri.emission;
            origin += t * direction;
            direction = sample_hemisphere(get_random_numbers(seed), tri.normal);
            throughput_weight *= tri.color * 2.0 * dot(tri.normal, direction);
        }
        else
            break;
    }
    return radiance;
}
```

The calling function:
```glsl
void mainImage(out vec4 out_color, in vec2 pixel_coord) {
    // Define the camera position and the view plane
    // Compute the camera ray
    // Use a different seed for each pixel and each frame

    // Perform path tracing with SAMPLE_COUNT paths
    out_color.rgb = vec3(0.0);
    for (int i = 0; i != SAMPLE_COUNT; ++i)
        out_color.rgb += get_ray_radiance(camera_position, ray_direction, seed);
    out_color.rgb /= float(SAMPLE_COUNT);
    // ...
}
```

# Progressive rendering on Shadertoy
```glsl
vec3 prev_color = texture(iChannel0, tex_coord).rgb;
float weight = 1.0 / float(iFrame + 1);
out_color.rgb = (1.0 - weight) * prev_color + weight * out_color.rgb;
out_color.a = 1.0;
```

TBC












