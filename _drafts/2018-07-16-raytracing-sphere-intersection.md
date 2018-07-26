---
title: Raytracing - Sphere Intersection
date: 2018-07-16
tags:
- Computer Graphics
- Raytracing
- Ray Tracing in One Weekend
- Math
- PBR
---
<img src="{{ site.url }}/images/glsl-smallpt-1.png" width="480"  style="display:block; margin:auto;">
In [Smallpt](http://www.kevinbeason.com/smallpt/) we only implemented [ray-sphere intersection](https://en.wikipedia.org/wiki/Line%E2%80%93sphere_intersection) for simplicity purpose. The [Cornell Box](http://www.graphics.cornell.edu/online/box/) floor ceiling and walls in this case are actually extremely big [spheres](https://en.wikipedia.org/wiki/Sphere).

# Line–sphere intersection
<!-- <img src="https://upload.wikimedia.org/wikipedia/commons/6/67/Line-Sphere_Intersection_Cropped.png" width="640"  style="display:block; margin:auto;">
The three possible line-sphere intersections: -->
Only 3 different ways:

1. No intersection.
2. Point intersection.
3. Two point intersection.

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

## Code
```c
// Ray–sphere intersection
float intersect(Sphere s, Ray r) {
    vec3 op = r.o - s.p; // (o-c), o: ray origin, c: sphere center position
    float half_b = dot(r.d, op); // b/2=l(o-c), l: ray direction
    float det = half_b * half_b - dot(op, op) + s.r * s.r;
    // determinant, det=(b/2)^2-(o-c)^2+r^2, simplified because l is unit vector
    float sqrt_det;
    float t;
    float epsilon = 1e-3; // maximum error value
    if (det < 0.)
        // ray missed sphere
        return 0.;
    else
        // det >= 0.
        // interseted or tangent
        sqrt_det = sqrt(det);

    // calculate final t
    // t=-b/2+sqrt(det) or t=-b/2-sqrt(det)
    t = -half_b - sqrt_det;

    // return smaller positive t, see note
    // 1. if-else style
    if(t>epsilon){
        return t;
    }
    else{
        t = -half_b + sqrt_det;
        if(t>epsilon){
            return t;
        }
        else{
            return 0.;
        }
    }
    // 2. more concise ternary operator style
    // return (t = b - det) > epsilon ? t : ((t = b + det) > epsilon ? t : 0.);
}
```

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

<!-- # Line–plane intersection
[TBC](https://en.wikipedia.org/wiki/Line%E2%80%93plane_intersection)
# Line-box intersection
[TBC](https://www.scratchapixel.com/lessons/3d-basic-rendering/minimal-ray-tracer-rendering-simple-shapes/ray-box-intersection) -->