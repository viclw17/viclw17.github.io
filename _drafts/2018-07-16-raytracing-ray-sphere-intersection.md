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
## Note
<img src="https://www.scratchapixel.com/images/upload/ray-simple-shapes/rayspherecases.png" width="640"  style="display:block; margin:auto;">
<div style="text-align:center">
<a href="https://www.scratchapixel.com/lessons/3d-basic-rendering/minimal-ray-tracer-rendering-simple-shapes/ray-sphere-intersection">click for source</a>
</div>
<br>

- If both $t$ are positive, ray is facing the sphere
- If one positive one negative, ray is shooting from inside
- If both $t$ are negative, ray is shooting away from the sphere

So we have to return smaller positive $t$ as the intersecting distance for the ray.

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
In analytic geometry, a sphere with center $(x_{0}, y_{0}, z_{0})$ and radius r is the _locus_ of all points $(x, y, z)$ such that

$$(x-x_{0})^{2}+(y-y_{0})^{2}+(z-z_{0})^{2}=r^{2}.$$

write in the form of vector we get

$$ \left\Vert {P}-{C}\right\Vert ^{2}=r^{2} $$

${P}$ is the point on the sphere, ${C}$ is sphere center point $(x_{0}, y_{0}, z_{0})$ and ${r}$ is sphere radius.



# Ray–sphere intersection

If we are talking about [line-sphere intersection](https://en.wikipedia.org/wiki/Line%E2%80%93sphere_intersection), mathematically there are only 3 different ways:

1. No intersection.
2. One point intersection, aka tangent.
3. Two point intersection.

But ray has origin and direction,

## Equation for a sphere:

$$ \left\Vert {\mathbf  {x}}-{\mathbf  {c}}\right\Vert ^{2}=r^{2} $$

<!-- - \(\mathbf {x}\)  - points on the sphere
- \(\mathbf {c}\)  - center point
- \({r}\) - radius -->

- $\mathbf {x}$ - points on the sphere
- $\mathbf {c}$ - center point
- ${r}$ - radius

## Equation for a line/ray starting at $\mathbf{o}$ :

$${\mathbf  {x}}={\mathbf  {o}}+d{\mathbf  {l}}$$

- $\mathbf {x}$ - **points** on the line
- $\mathbf{o}$ - **origin** of the line
- $d$ - **distance** along line from starting point
- ${\mathbf {l}}$ - **direction** of line (a unit vector)

## Solve intersection:
Searching for points that are on the line and on the sphere means combining the equations and solving for $d$ :

Equations combined (联立方程组), then expanded and rearranged:

$$\left\Vert {\mathbf  {o}}+d{\mathbf  {l}}-{\mathbf  {c}}\right\Vert ^{2}=r^{2}\Leftrightarrow d^{2}({\mathbf  {l}}\cdot {\mathbf  {l}})+2d({\mathbf  {l}}\cdot ({\mathbf  {o}}-{\mathbf  {c}}))+({\mathbf  {o}}-{\mathbf  {c}})\cdot ({\mathbf  {o}}-{\mathbf  {c}})-r^{2}=0$$

The form of a quadratic formula is now observable:

$$ad^{2}+bd+c=0$$

where:

- $a={\mathbf  {l}}\cdot {\mathbf  {l}}=\left\Vert {\mathbf  {l}}\right\Vert ^{2}$
- $b=2({\mathbf  {l}}\cdot ({\mathbf  {o}}-{\mathbf  {c}}))$
- $c=({\mathbf  {o}}-{\mathbf  {c}})\cdot ({\mathbf  {o}}-{\mathbf  {c}})-r^{2}=\left\Vert {\mathbf  {o}}-{\mathbf  {c}}\right\Vert ^{2}-r^{2}$

in quadratic formula $ax^{2}+bx+c=0$, the solution is $x={\frac {-b\pm {\sqrt {b^{2}-4ac}}}{2a}}$, and also because $a = \left\Vert {\mathbf  {l}}\right\Vert ^{2}=1$ (unit vector), so:

$$d=-({\mathbf  {l}}\cdot ({\mathbf  {o}}-{\mathbf  {c}}))\pm {\sqrt  {({\mathbf  {l}}\cdot ({\mathbf  {o}}-{\mathbf  {c}}))^{2}-\left\Vert {\mathbf  {o}}-{\mathbf  {c}}\right\Vert ^{2}+r^{2}}} $$

$$det=({\mathbf  {l}}\cdot ({\mathbf  {o}}-{\mathbf  {c}}))^{2}-\left\Vert {\mathbf  {o}}-{\mathbf  {c}}\right\Vert ^{2}+r^{2}$$

- If $det$ < 0, the line does not intersect the sphere;
- If $det$ = 0, the line just touches the sphere in one point (tangent);
- If $det$ > 0, the line touches the sphere in two points (intersected).

Sphere: dot((p-c),(p-c))=R*R --> any point p that satisfies this equation is on the sphere.

Ray: p(t) = A + t*B

dot((p(t)-c),(p(t)-c))=R*R
dot((A + t*B - c),(A + t*B - c))=R*R
t*t*dot(B,B) + 2*t*dot(B,A-C) + dot(A-C,A-C) - R*R = 0
a = dot(B,B)
b = 2*dot(B,A-C)
c = dot(A-C,A-C) - R*R

``` c
float hit_sphere(const vec3& center, float radius, const ray& r){
    vec3 oc = r.origin() - center; // A-C
    float a = dot(r.direction(), r.direction());
    float b = 2.0 * dot(oc, r.direction());
    float c = dot(oc,oc) - radius*radius;
    float discriminant = b*b - 4*a*c;
    // return (discriminant>0);
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
