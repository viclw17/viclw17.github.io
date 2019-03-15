# Project H2O
## bubble vfx
1. mesh particle + opaque relection + displacement material
2. quads + transparency + refraction material
## watersplash
subsurface unlit
## buoyancy
world offset
## title fx
uv coord distortion
## fish particle
uv mask/ displacement
spline BP
![test](images/Work/fish.gif)

---

## Exploration of Interaction Effect in VR
During my current VR project, I was trying to create an effect to "soften out" the harsh clipping when players interact with the environment. This problem has been well addressed by [this amazing post](http://blog.leapmotion.com/interaction-sprint-exploring-the-hand-object-boundary/).

<img src="http://blog.leapmotion.com/wp-content/uploads/2017/12/standard-clipping.gif" width="480" style="display:block; margin:auto;">
<br>
<figcaption style="text-align: center;">Image from - <a href="http://blog.leapmotion.com/interaction-sprint-exploring-the-hand-object-boundary">blog.leapmotion.com</a></figcaption>
<br>

In VR development, because the motion of the first person character is precisely mapping the real players' body(mainly hands) movement, the environment collision will never be able to prevent the player model from intersecting with the surrounding meshed. This is a well-accepted limitation of VR technique, however I personally feel it is a bit annoying if there is nothing done visually to address this fact.

The above linked post provides several solutions to this issue and are all
very practical and effective. There are mainly 3 ways, according to the post, to create a _hands intersection effect_:

1. Use **depth fade** to highlight the intersection;
2. Use texture mask to highlight the finger tips;
3. Add responsive meshes at the intersecting points.

Here in the post I want to cover more details about the implementation and troubleshooting of the first 2 solutions.

### Depth Fade for Intersection
<img src="http://blog.leapmotion.com/wp-content/uploads/2017/12/intersection-shader.gif" width="480" style="display:block; margin:auto;">
<br>
<figcaption style="text-align: center;">Image from - <a href="http://blog.leapmotion.com/interaction-sprint-exploring-the-hand-object-boundary">blog.leapmotion.com</a></figcaption>
<br>

Unreal's [depth expressions](https://docs.unrealengine.com/en-us/Engine/Rendering/Materials/ExpressionReference/Depth) are extremely simple yet versatile on creating effects like intersection highlight, enegery field/shield, scanning field etc.

<!-- <img src="https://i.ytimg.com/vi/Dw8v4UYZcjA/maxresdefault.jpg" width="480" style="display:block; margin:auto;">
<br>
<figcaption style="text-align: center;">Image from - <a href="https://www.corbuzier.tk/2017/11/ue4-holo-shield-makingof/kG6Xt1NDgMi.html">Source</a></figcaption>
<br> -->

<img src="{{ site.url }}/images/Work/depthfade.gif" width="640" style="display:block; margin:auto;">
<br>






## emi hand fx
fix self occlusion
https://www.tomlooman.com/the-many-uses-of-custom-depth-in-unreal-4/
https://docs.unrealengine.com/en-us/Engine/Rendering/Materials/ExpressionReference/Depth

## emi postprocessing fx
## emi attack fx
## emi vertex emiting particles
## algae localized fx
## glass window
glass material
physxlab destruction workflow
## glowstick
## hologram
## ending title sequence UMG
## rain fx
## underwater explosion
