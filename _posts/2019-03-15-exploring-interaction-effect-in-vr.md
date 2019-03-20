---
title: Exploring Interaction Effect in VR
date: 2019-03-15
tags:
- Unreal Engine
- Shader
---
In VR development, because the motion of the first person character is precisely mapping the real players' body(mainly hands) movement, the environment collision will never be able to prevent the player model from intersecting with the surrounding meshed. This is a well-accepted limitation of VR technique, however I personally feel it is a bit annoying if there is nothing done visually to address this fact.

During my current VR project, I was trying to create an effect to "soften out" the harsh clipping when players interact with the environment. This problem has been well addressed by [this amazing post](http://blog.leapmotion.com/interaction-sprint-exploring-the-hand-object-boundary/).

<img src="http://blog.leapmotion.com/wp-content/uploads/2017/12/standard-clipping.gif" width="480" style="display:block; margin:auto;">
<figcaption style="text-align: center;">Image from - <a href="http://blog.leapmotion.com/interaction-sprint-exploring-the-hand-object-boundary">blog.leapmotion.com</a></figcaption>
<br>

The post provides several solutions to this issue and are all
very practical and effective:

1. Use Depth Expressions to highlight the intersection;
2. Use texture mask to highlight the finger tips;
3. Add responsive meshes at the intersecting points.

Here in the post I want to cover more details about the implementation and troubleshooting of the **first 2 solutions**.

# Use Depth Expressions to Highlight the Intersection
<!-- <img src="http://blog.leapmotion.com/wp-content/uploads/2017/12/intersection-shader.gif" width="480" style="display:block; margin:auto;">
<figcaption style="text-align: center;">Image from - <a href="http://blog.leapmotion.com/interaction-sprint-exploring-the-hand-object-boundary">blog.leapmotion.com</a></figcaption> -->

## Unreal Depth Expressions
Unreal's [depth expressions](https://docs.unrealengine.com/en-us/Engine/Rendering/Materials/ExpressionReference/Depth) are extremely simple to use yet very versatile on creating effects like intersection highlight, enegery field/shield, scanning field etc.

<img src="{{ site.url }}/images/Work/depth_expression.jpg" width="480" style="display:block; margin:auto;">

### Pixel Depth
The ```PixelDepth``` expression outputs the **distance** from the camera to the pixels currently being rendered.

The values it returns are very big and not normalized, so if you plug it into color output the result will be all blown out. You have to devide it with a big number and clamp it to get a proper look.

<img src="{{ site.url }}/images/Work/depth_pixel1.gif" width="480" style="display:block; margin:auto;">
<figcaption style="text-align: center;">Output as color.</figcaption>
<br>
<img src="{{ site.url }}/images/Work/depth_pixel2.gif" width="480" style="display:block; margin:auto;">
<figcaption style="text-align: center;">Use it to drive opacity.</figcaption>
<br>

### Scene Depth
The ```SceneDepth``` expression outputs the existing scene depth. This is similar to ```PixelDepth```, except that ```PixelDepth``` can sample the depth only at the pixel currently being rendered, whereas ```SceneDepth``` can sample depth **at any location**.

If an object has material outputing ```SceneDepth``` as color (after scaling and clamping), because of this feature, the output can show the objects that occluded. Also because it's accessing the scene depth, the material using this expression should use translucent as its blend mode to avoid itself writing the depth buffer.

> Only translucent materials may utilize ```SceneDepth```!

<img src="{{ site.url }}/images/Work/depth_visualization2.gif" width="480" style="display:block; margin:auto;">
<figcaption style="text-align: center;">Sphere in the front assigned with material that outputs ```SceneDepth``` as color; sphere in the upper back is in translucent blend mode, which is not written into scene depth buffer.</figcaption>
<br>

### Depth Fade
```DepthFade``` is a packed-up node used to fade a transparent object according to scene depth pass, to soften the seam where it intersects with opaque object.

<img src="{{ site.url }}/images/Work/depth_fade1.gif" width="480" style="display:block; margin:auto;">
<figcaption style="text-align: center;">Output as color.</figcaption>
<br>
<img src="{{ site.url }}/images/Work/depth_fade2.gif" width="480" style="display:block; margin:auto;">
<figcaption style="text-align: center;">Use it to drive opacity.</figcaption>
<br>
<img src="{{ site.url }}/images/Work/depth_fade3.gif" width="480" style="display:block; margin:auto;">
<figcaption style="text-align: center;">Use it to highlight the geometry intersection. (can also make it two-sided for better effect.)</figcaption>
<br>

Combined with other tricks, this node can help to achieve all sorts of cool FX like energy fence/shield etc.

<img src="{{ site.url }}/images/Work/depth_fade_fx.jpg" width="480" style="display:block; margin:auto;">
<br>
<img src="{{ site.url }}/images/Work/depth_fade4.gif" width="480" style="display:block; margin:auto;">
<figcaption style="text-align: center;">Add tiling noise to the intersection.</figcaption>
<br>

<!-- <img src="https://i.ytimg.com/vi/Dw8v4UYZcjA/maxresdefault.jpg" width="480" style="display:block; margin:auto;">
<br>
<figcaption style="text-align: center;">Image from - <a href="https://www.corbuzier.tk/2017/11/ue4-holo-shield-makingof/kG6Xt1NDgMi.html">Source</a></figcaption>
<br> -->

## Creating the intersection hightlight
After going through all the above concept, let's create the intersection hightlight on a hand. Just simply use ```DepthFade``` node to lerp the emissive output.

<img src="{{ site.url }}/images/Work/depth_fade_highlight.jpg" width="480" style="display:block; margin:auto;">
<br>
However, this causes issues that because ```DepthFade``` requires material to be translucent, the final rendering has serious lighting and sorting problems. The sorting issue is a common problem to encounter while working with transparency in Unreal, and is also a known limitation of the engine. [Tom Looman's Amazing Post](https://www.tomlooman.com/the-many-uses-of-custom-depth-in-unreal-4/) explains the solution very well and here I am going to document my way of implementation for future reference.

To fix the translucent material lighting problem, just change the **translucency lighting mode** to **Surface ForwardShading**.

<img src="{{ site.url }}/images/Work/lighting.jpg" width="480" style="display:block; margin:auto;">
<br>
<img src="{{ site.url }}/images/Work/depth_bug3.jpg" width="480" style="display:block; margin:auto;">
<figcaption style="text-align: center;">Opaque material/ translucent material with default lighting mode/ translucent material with ForwardShading lighting mode</figcaption>
<br>
<img src="{{ site.url }}/images/Work/depth-bug1.gif" width="480" style="display:block; margin:auto;">

Now the lighting is correct but we still have ugly self-sorting bug. To fix this problem, we need to use a little trick with **custom depth buffer**. Basically you can label certain mesh components to be written into custom depth buffer and later to be accessed in material editor through ```SceneTexture:CustomDepth``` node.

Since the engine itself cannot handle self-sorting for translucent objects, we can calculate by ourselves. Just create a duplicated dummy mesh and set it to be written into custom depth buffer but **not rendered in the main pass**.

> I also tend to parent the dummy mesh under the original mesh and name it "DepthFix".

<img src="{{ site.url }}/images/Work/depth_fix.jpg" width="480" style="display:block; margin:auto;">
<figcaption style="text-align: center;"></figcaption>

You can visualize the custom buffer in the editor, and then you will see the dummy mesh got rendered there.
<img src="{{ site.url }}/images/Work/custom_depth.jpg" width="480" style="display:block; margin:auto;">
<figcaption style="text-align: center;"></figcaption>
<br>
<img src="{{ site.url }}/images/Work/custom_depth2.jpg" width="480" style="display:block; margin:auto;">

Next, in the material, calculate transparency based on ```PixelDepth``` and ```CustomDepth``` and output into ```Opacity```.
<img src="{{ site.url }}/images/Work/depth_fade_fix.jpg" width="480" style="display:block; margin:auto;">
<br>
<img src="{{ site.url }}/images/Work/depth_bug2.gif" width="480" style="display:block; margin:auto;">
<figcaption style="text-align: center;">Playing with different DepthBias and Opacity.</figcaption>
<br>

Now with all the work above, we have a nice intersection highlight effect that
1. Use ```DepthFade``` to get the intersection
2. Use ```ForwardShading``` lighting mode to fix the translucent lighting problem
3. Use dummy mesh and ```CustomDepth``` to fix the self-sorting problem

> Note that any operation you did with the original mesh, like play animation, you have to apply to the dummy mesh as well to keep both of them in sync!

<img src="{{ site.url }}/images/Work/final.gif" width="480" style="display:block; margin:auto;">

TBC
