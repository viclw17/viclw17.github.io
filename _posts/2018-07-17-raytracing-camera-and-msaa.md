---
title: Raytracing - Camera and Multisampling Antialiasing
date: 2018-07-17
tags:
- Computer Graphics
- Raytracing
- Ray Tracing in One Weekend
- PBR
---
<img src="{{ site.url }}/images/raytracing-camera-msaa-01.jpg" width="480"  style="display:block; margin:auto;">
<br>
Finish reading [Ray Tracing in One Weekend](http://in1weekend.blogspot.com/2016/01/ray-tracing-in-one-weekend.html) (RTIOW) Chapter 2, 3 and 4. Raytracing analogy, simple camera model and MSAA implementation.

# Raytracing Overview
**Light Rays** (photons) are emitted from/bounced by/passing through the objects, and some of them made their way to arrive our eye retina/camera film to form the image we see/capture.

To reverse-engineer this, imaging from view origin (camera/eye position) we are shooting(emitting) **View Rays** through **every pixel** to "scan" the objects we are trying to render, and then gather the **hit points** to color the corresponding pixels.

<!-- the core of a raytracer is to send rays through pixels and compute what color is seen in the direction of those rays. This is of the form calculate which ray goes from the eye to a pixel, compute what that ray intersects, and compute a color for that intersection point. -->

<img src="http://www.scratchapixel.com/images/upload/ray-tracing-refresher/rt-setup2.png" width="480"  style="display:block; margin:auto;">
<br>


# Draw with Ray
## Simple Camera Model
The variation of the camera ray directions (one per pixel) form a "view volume/cone" - the **view frustum**, and every single ray provides a color value to its corresponding pixel to form the final image on a "film/retina" - the **near clipping plane**.

<!-- <img src="https://www.techjay.com/wp-content/uploads/2017/10/film-projector-lens-and-light-footage-024706786_prevstill.jpeg" width="480"  style="display:block; margin:auto;">
<br> -->

<img src="http://www.falloutsoftware.com/tutorials/gl/perspective-transform-visual-diagram-opengl-3d-to-2d.png" width="480"  style="display:block; margin:auto;">
<br>

So to draw an image with raytracing, we need to:
1. Set a range of the directions of the camera rays. This define the size of the screen.
2. Set color value for the pixel of every rays. (for higher rendering quality we can shoot multiple rays per pixel and average the coloration for that pixel.)

In order to respect the convention of a **right-handed coordinate system**, the negative z-axis is pointing into the screen.
>For right-handed coordinates your right thumb points along the Z axis in the positive direction and the curl of your fingers represents a motion from the first or X axis to the second or Y axis. When viewed from the top or Z axis the system is counter-clockwise.

<img src="https://upload.wikimedia.org/wikipedia/commons/e/e2/Cartesian_coordinate_system_handedness.svg" width="320"  style="display:block; margin:auto;">
<div style="text-align:center">
Left-handed coordinates on the left Right-handed coordinates on the right.
</div>
<br>

Here we set the near clipping plane at -1 on the z-axis, and define the size of the screen with range from -2 to 2 on x-axis , and from -1 to 1 on y-axis.

```c
class camera {
public:
    camera() {
        lower_left_corner = vec3(-2.0,-1.0,-1.0);
        horizontal = vec3(4.0,0.0,0.0); // horizontal range
        vertical = vec3(0.0,2.0,0.0);   // vertical range
        origin = vec3(0.0,0.0,0.0);
    }

    ray get_ray(float u, float v){return ray(origin, lower_left_corner + u*horizontal + v* vertical - origin);}

    vec3 lower_left_corner;
    vec3 horizontal;
    vec3 vertical;
    vec3 origin;
};
```
And we can build the camera rays with **camera position** and the **pixel positions** on the clipping plane. After we have the rays, we can use it to trace the scene and produce color values to draw the final image.

<img src="{{ site.url }}/images/raytracing-camera-msaa-figure.PNG" width="720"  style="display:block; margin:auto;">
<br>

```c
// pixel amount on x and y axis
int nx = 200;
int ny = 100;

camera cam;

for (int j = ny-1; j >= 0; j--)
{
    for (int i = 0; i < nx; i++)
    {
        vec3 col(0,0,0);

        float u = float(i) / float(nx);
        float v = float(j) / float(ny);

        ray r = cam.get_ray(u,v);
        vec3 p = r.point_at_parameter(1.0);
        p = unit_vector(p);
        col = vec3(p.z(),p.z(),p.z());

        int ir = int (255.99*col[0]);
        int ig = int (255.99*col[1]);
        int ib = int (255.99*col[2]);
    }
}
```

<img src="{{ site.url }}/images/raytracing-camera-msaa-1.jpg" width="320"  style="display:block; margin:auto;">
<div style="text-align:center">
Color with depth values of normalized camera rays.
</div>

## Draw Sphere
```c
bool hit_sphere(const vec3& center, float radius, const ray& r) {
    vec3 oc = r.origin() - center;
    float a = dot(r.direction(), r.direction());
    float b = 2.0 * dot(oc, r.direction());
    float c = dot(oc,oc) - radius*radius;
    float discriminant = b*b - 4*a*c;
    return (discriminant>0);
}

vec3 color(const ray& r) {
    // draw blue sphere
    if(hit_sphere(vec3(0,0,-1), 0.5, r))
        return vec3(0,0,1);

    // draw background
    vec3 unit_direction = unit_vector(r.direction());
    float t = 0.5*(unit_direction.y()+1.0); // -1~1 --> 0~1
    return (1.0-t)*vec3(1.0,1.0,1.0) + t*vec3(0.5,0.7,1.0); // lerp
}

int main() {
    int nx = 200;
    int ny = 100;

    camera cam;

    for (int j = ny-1; j >= 0; j--) {
        for (int i = 0; i < nx; i++) {
            vec3 col(0,0,0);

            float u = float(i) / float(nx);
            float v = float(j) / float(ny);
            ray r = cam.get_ray(u,v);
            vec3 col = color(r);

            int ir = int (255.99*col[0]);
            int ig = int (255.99*col[1]);
            int ib = int (255.99*col[2]);
        }
    }
}
```
```blended_value = (1-t)*start_value + t*end_value``` is using **linear interpolation** to create a gradient background.

<img src="{{ site.url }}/images/raytracing-camera-msaa-2.jpg" width="320"  style="display:block; margin:auto;">
<div style="text-align:center">
Draw a sphere with raytracing and color it blue, along with background gradient.
</div>
<!-- <br> -->

# Multisampling Antialiasing
We have been assigning **single color** value for every single pixel. This causes bad aliasing effect in the final image.

Solution is to randomly shoot multiple camera rays per pixel and get multiple hit result for coloration. The amount of random rays is **Sample Rate**. Then we add up all the various color values for this pixel and divide it by sample rate to average the final color. This color would be much more representative than the color provided by only one ray.

<!--
之前的图是每个像素点设置一个颜色值，相当于将像素中心位置的颜色设置给了整个像素。所以，如果两个像素点中心位置的颜色值相差比较大时，这两个像素点就会产生清晰的边界。那么问题来了，怎么使得两个像素点的边界模糊呢？
书上的做法是：针对每个像素点随机采样100次，获得100个颜色值，然后将这100个颜色值的平均值设置为整个像素点的颜色值。
-->

<img src="http://www.sidefx.com/docs/houdini/images/render/mug/sampling_tab/PixelSampling.jpg" width="480"  style="display:block; margin:auto;">
<br>

``` c
int nx = 200;
int ny = 100;
int ns = 100;

camera cam;

for(int j=ny-1; j>=0; j--){
    for(int i=0; i<nx; i++){
        vec3 col(0,0,0);
        for(int s=0; s < ns; s++){
            float u = float(i + drand48())/float(nx); // 0~1
            float v = float(j + drand48())/float(ny);
            ray r = cam.get_ray(u,v);
            col += color(r);
        }
        col /= float(ns);

        int ir = int(255.99*col[0]);
        int ig = int(255.99*col[1]);
        int ib = int(255.99*col[2]);
    }
}
```

<img src="{{ site.url }}/images/raytracing-camera-msaa-3.jpg" width="640"  style="display:block; margin:auto;">
<div style="text-align:center">
w/o and w/ MSAA
</div>
<br>
<img src="{{ site.url }}/images/raytracing-camera-msaa-4-0.jpg" width="640"  style="display:block; margin:auto;">
<div style="text-align:center">
MSAA off, 1 sample per pixel
</div>
<br>
<img src="{{ site.url }}/images/raytracing-camera-msaa-4-10.jpg" width="640"  style="display:block; margin:auto;">
<div style="text-align:center">
MSAA on, 10 sample per pixel
</div>
<br>
<img src="{{ site.url }}/images/raytracing-camera-msaa-4-50.jpg" width="640"  style="display:block; margin:auto;">
<div style="text-align:center">
MSAA on, 50 sample per pixel
</div>
<br>
