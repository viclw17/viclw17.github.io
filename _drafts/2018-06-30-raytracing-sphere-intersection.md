---
title: Raytracing - Ray-Geometry Intersection
date: 2018-06-30
tags:
- Computer Graphics
- Raytracing
- Math
- PBR
---
<img src="http://www.kevinbeason.com/smallpt/result_25k.png" width="320"  style="display:block; margin:auto;">
Apart from implementing all the fancy raytracing algorithm, we have to firstly define the basic geometries and build a simple scene to render. [Smallpt](http://www.kevinbeason.com/smallpt/) is using a modified [Cornell Box](http://www.graphics.cornell.edu/online/box/) as the scene. We also have to implement the calculation of line-geometry intersection to get the information of the point where the ray hit on the objects in the scene.

# Sphere Equations
## Analytic Form
In analytic geometry, a sphere with center (x0, y0, z0) and radius r is the locus of all points (x, y, z) such that

$$(x-x_{0})^{2}+(y-y_{0})^{2}+(z-z_{0})^{2}=r^{2}$$

Expand and rearrange the equation to turn it into a function:

$$f(x,y,z)=a(x^{2}+y^{2}+z^{2})+2(bx+cy+dz)+e$$

where $a, b, c, d, e$ are real numbers and $a ≠ 0$, and we have

$$x_{0}={\frac {-b}{a}},\quad y_{0}={\frac {-c}{a}},\quad z_{0}={\frac {-d}{a}},\quad \rho ={\frac {b^{2}+c^{2}+d^{2}-ae}{a^{2}}}.$$

Let the function equal to 0, $f(x,y,z)=0$
- If $\rho < 0$, the equation has no real solutions, and is called the equation of an imaginary sphere.
- If $\rho = 0$, the equation has only one solution at point $P_{0}=(x_{0},y_{0},z_{0})$, and is called the equation of a point sphere.
- If $\rho > 0$, the equation is a sphere whose center is $P_{0}$ and radius is ${\sqrt {\rho }}$.

If $a$ in the above equation is $0$ then $f(x, y, z) = 0$ is the equation of a **plane**. Thus, a plane may be thought of as **a sphere of infinite radius whose center is a point at infinity**.

## Parameterized Form*
<img src="https://upload.wikimedia.org/wikipedia/commons/5/51/Spherical_Coordinates_%28Colatitude%2C_Longitude%29.svg" width="240"  style="display:block; margin:auto;">
<br>

The points on the sphere with radius $r > 0$ and center $(x_{0},y_{0},z_{0})$ can be parameterized via

$${\begin{aligned}x&=x_{0}+r\sin \varphi \;\cos \theta \\y&=y_{0}+r\sin \varphi \;\sin \theta \qquad (-{\pi }/2\leq \varphi \leq {\pi }/2,\;0\leq \theta <2\pi )\\z&=z_{0}+r\cos \varphi \,\end{aligned}}$$

## Differential Form**
A sphere of any radius centered at zero is an integral surface of the following differential form:

$$x\,dx+y\,dy+z\,dz=0$$

This equation reflects that **position and velocity vectors** of a point, $(x, y, z)$ and $(dx, dy, dz)$, traveling on the sphere are **always orthogonal to each other**.

# Scene Setup
>Note that in [Smallpt](http://www.kevinbeason.com/smallpt/) it only implemented [ray-sphere intersection](https://en.wikipedia.org/wiki/Line%E2%80%93sphere_intersection) for simplicity purpose. The [Cornell Box](http://www.graphics.cornell.edu/online/box/) floor ceiling and walls in this case are actually extremely big spheres.

## Define ```struct```
```c
struct Ray{
    vec3 origin;
    vec3 dir;
};
struct Material{
    int refl;
    vec3 emission;
    vec3 color;
    float ior;
};
struct Sphere{
    float radius;
    vec3 pos;
    Material mat;
};
struct Plane{
    vec3 pos;
    vec3 normal;
    Material mat;
};
```
## Scene Initialization ```initScene()```
```c
// NOTE(Victor): Scene Description
#define NUM_SPHERES 3
#define NUM_PLANES 6
Sphere spheres[NUM_SPHERES];
Plane planes[NUM_PLANES];
void initScene() {
    spheres[0] = Sphere(16.5, vec3(27, 16.5, 47),  Material(DIFF, vec3(0.), vec3(1.), 0.));
    spheres[1] = Sphere(16.5, vec3(73, 16.5, 78),  Material(DIFF, vec3(0.), vec3(.75, 1., .75), 1.5));
    spheres[2] = Sphere(600., vec3(50, 689.3, 50), Material(DIFF, vec3(6.), vec3(0.), 0.));
    planes[0] = Plane(vec3(0, 0, 0),   vec3(0, 1, 0),  Material(DIFF, vec3(0.), vec3(.75), 0.));
    planes[1] = Plane(vec3(-7, 0, 0),  vec3(1, 0, 0),  Material(DIFF, vec3(0.), vec3(.75, .25, .25), 0.));
    planes[2] = Plane(vec3(0, 0, 0),   vec3(0, 0, -1), Material(DIFF, vec3(0.), vec3(.75), 0.));
    planes[3] = Plane(vec3(107, 0, 0), vec3(-1, 0, 0), Material(DIFF, vec3(0.), vec3(.25, .25, .75), 0.));
    planes[4] = Plane(vec3(0, 0, 180), vec3(0, 0, 1),  Material(DIFF, vec3(0.), vec3(0.), 0.));
    planes[5] = Plane(vec3(0, 90, 0),  vec3(0, -1, 0), Material(DIFF, vec3(0.), vec3(.75), 0.));
}
```

# Line–Sphere Intersection
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

## Consider A Ray
In the case of raytracing, we are dealing with a vector(ray) instead of a line. A ray has
<img src="https://www.scratchapixel.com/images/upload/ray-simple-shapes/rayspherecases.png" width="640"  style="display:block; margin:auto;">
<div style="text-align:center">
<a href="https://www.scratchapixel.com/lessons/3d-basic-rendering/minimal-ray-tracer-rendering-simple-shapes/ray-sphere-intersection">click for source</a>
</div>
<br>

- If both $t$ are positive, ray is facing the sphere
- If one positive one negative, ray is shooting from inside
- If both $t$ are negative, ray is shooting away from the sphere

So we have to return smaller positive $t$ as the intersecting distance for the ray.

## Ray–Sphere Intersection Code
```c
// Ray–sphere intersection
float intersectSphere(Sphere s, Ray r) {
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



<!-- # Line–plane intersection
[TBC](https://en.wikipedia.org/wiki/Line%E2%80%93plane_intersection)
# Line-box intersection
[TBC](https://www.scratchapixel.com/lessons/3d-basic-rendering/minimal-ray-tracer-rendering-simple-shapes/ray-box-intersection) -->

```c
float intersectPlane(Ray r, Plane p) {
    float t = dot(p.pos - r.origin, p.normal) / dot(r.dir, p.normal);
    return mix(0., t, float(t > EPSILON));
}
```
