---
title: Raytracing - Ray Sphere Intersection
date: 2018-07-16
tags:
- Computer Graphics
- Raytracing
- Ray Tracing in One Weekend
- Math
- PBR
---
<!-- <img src="https://farm5.staticflickr.com/4342/36550535442_3b9da55913_z.jpg" width="640" hight="480" style="display:block; margin:auto;"> -->
<!-- <img src="https://upload.wikimedia.org/wikipedia/commons/3/32/Recursive_raytrace_of_a_sphere.png" width="480" hight="480" style="display:block; margin:auto;"> -->
In raytracer, calculating ray - object intersection is very important on locating the hit point and producing correct color for the corresponding pixel. Sphere is always the best geometrical shape to start with as it is one of the simplest shape to describe mathematically. In this post I documented typical ```Ray``` class definition, ray-sphere intersection math breakdown, and code implementation. I will keep updating the post for solutions of other types of shapes.

# Ray
A ray has an origin (light source) and a direction (light direction). Ray can be described mathematically as

$$P(t)=A+tB$$

$P$ is the point on the ray. $A$ is the origin of the ray. $B$ is the direction of the ray which is **a unit vector**. $t$ is a parameter used to move $P$ away from $A$ on the direction of $B$. Thus, $P$ can be located just using $t$ so we used the notation $P(t)$ to make it look like a function.

<img src="https://qph.fs.quoracdn.net/main-qimg-8f765bf77d8cdc7332d4acfc04f6e94f" width="200"  style="display:block; margin:auto;">

Unlike line, ray has a start point and travel direction. So we define when $t>0$ the ray is travelling toward its **forward direction**.

## Code
``` c
#include "vec3.h"
class ray {
    public:
        ray() {}
        ray(const vec3& a, const vec3& b) { A = a; B = b; }
        vec3 orgin() const     { return A; }
        vec3 direction() const { return B; }
        vec3 point_at_parameter(float t) const { return A + t*B; }

        vec3 A;
        vec3 B;
};
```

# Sphere
In analytic geometry, a sphere with center $(x_{0}, y_{0}, z_{0})$ and radius r is the [_locus_](https://en.wikipedia.org/wiki/Locus_(mathematics)) of all points $(x, y, z)$ such that

$$(x-x_{0})^{2}+(y-y_{0})^{2}+(z-z_{0})^{2}=r^{2}.$$

write in the form of vector we get

$$ \left\Vert {P}-{C}\right\Vert ^{2}=r^{2} $$

which is an equivalent of $dot((P-C),(P-C))=r^{2}$ where ${P}$ is the point on the sphere, ${C}$ is sphere center point $(x_{0}, y_{0}, z_{0})$ and ${r}$ is sphere radius.

# Ray–Sphere Intersection
When the ray and sphere intersect, the intersection points are shared by both equations. Searching for points that are on the ray and on the sphere means combining the equations and solving for $t$.

- Sphere: $dot((P-C),(P-C))=r^{2}$
- Ray: $p(t) = A + tB$
- Combined: $dot((A + tB - C),(A + tB - C))=r^{2}$

then expanded and rearranged:

$$t^{2}\cdot dot(B,B)+2t \cdot dot(B, A-C)+dot(A-C,A-C)-r^{2}=0$$

The form of a quadratic equation is now observable:

$$at^{2}+bt+c=0$$

where:

- $a = dot(B,B)$
- $b = 2\cdot dot(B,A-C)$
- $c = dot(A-C,A-C) - r^{2}$

With the above parameterization, the quadratic formula is:

$$ t={\frac {-b\pm {\sqrt {b^{2}-4ac}}}{2a}}$$

where $b^{2}-4ac$ is the **discriminant** of the equation, and

- If $discriminant < 0$, the line of the ray does not intersect the sphere (missed);
- If $discriminant = 0$, the line of the ray just touches the sphere in one point (tangent);
- If $discriminant > 0$, the line of the ray touches the sphere in two points (intersected).



``` c
bool hit_sphere(const vec3& center, float radius, const ray& r){
    vec3 oc = r.origin() - center;
    float a = dot(r.direction(), r.direction());
    float b = 2.0 * dot(oc, r.direction());
    float c = dot(oc,oc) - radius*radius;
    float discriminant = b*b - 4*a*c;
    return (discriminant>0);
}
```

# Intersection Distance
If we are talking about [line-sphere intersection](https://en.wikipedia.org/wiki/Line%E2%80%93sphere_intersection) mathematically, there are only 3 different ways:

1. No intersection.
2. One point intersection, aka tangent.
3. Two point intersection.

But ray has origin and direction, so there are more spacific scenarios:

<img src="https://www.scratchapixel.com/images/upload/ray-simple-shapes/rayspherecases.png" width="640"  style="display:block; margin:auto;">
<div style="text-align:center">
<a href="https://www.scratchapixel.com/lessons/3d-basic-rendering/minimal-ray-tracer-rendering-simple-shapes/ray-sphere-intersection">source</a>
</div>
<br>

1. No intersection.
2. One point intersection, aka tangent.
3. Two point intersection.
    - If both $t$ are positive, ray is facing the sphere and intersecting.
    - If one $t$ positive one $t$ negative, ray is shooting from inside.
    - If both $t$ are negative, ray is shooting away from the sphere, and techinically ray-sphere intersection is actually impossible.

So we have to return **smaller** and **positive** $t$ as the intersecting distance for the ray.

``` c
float hit_sphere(const vec3& center, float radius, const ray& r){
    vec3 oc = r.origin() - center;
    float a = dot(r.direction(), r.direction());
    float b = 2.0 * dot(oc, r.direction());
    float c = dot(oc,oc) - radius*radius;
    float discriminant = b*b - 4*a*c;
    if(discriminant < 0){
        return -1.0;
    }
    else{
        return (-b - sqrt(discriminant)) / (2.0*a);
    }
}
```

<!-- # Line–plane intersection
[TBC](https://en.wikipedia.org/wiki/Line%E2%80%93plane_intersection)
# Line-box intersection
[TBC](https://www.scratchapixel.com/lessons/3d-basic-rendering/minimal-ray-tracer-rendering-simple-shapes/ray-box-intersection) -->
