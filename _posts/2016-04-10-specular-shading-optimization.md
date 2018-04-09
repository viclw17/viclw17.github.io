---
title: Specular Shading Optimization For A More Natural Looking
date: 2016-04-10
tags:
- [Unity Shader]
- [Unity]
- [Shader]
- [Optimization]
---
<img src="{{ site.url }}/images/specular2.gif" width="400" height="400" style="display:block; margin:auto;">
<figcaption style="text-align: center;">Low poly water mesh with unity standard specular material.</figcaption>
<br />
When learning and practicing shader programming for Unity, I found the [Cg Wiki](https://en.wikibooks.org/wiki/Cg_Programming/Unity) website extremely benevolent on getting a well-rounded understanding of how to shade an object step by step. Apart from the magic configuration parts(Shader Properties/Passes/Tags etc.) of a typical Unity shader, the main function-related part of the shader only lies between _CGPROGRAM_ and _ENDCG_. According to my practice, I believe [Cg Wiki](https://en.wikibooks.org/wiki/Cg_Programming/Unity) and [Unity Shader Reference](http://docs.unity3d.com/Manual/SL-Reference.html) can cover almost all the basic tricks of Unity shader programming. ;)

However, when I went through the [Specular Highlights](https://en.wikibooks.org/wiki/Cg_Programming/Unity/Specular_Highlights) tutorial trying to achieve a [Phong Shading](https://en.wikipedia.org/wiki/Phong_shading) effect from scratch, I found the code snippet that the tutorial provided has some artifacts.

<img src="http://www.itchy-animation.co.uk/tutorials/01-intro-01.jpg" width="400" height="400" style="display:block; margin:auto;">
<figcaption style="text-align: center;">Different areas of a typical shaded sphere. <br /> Note that the terminator line should be a soft gradient. </figcaption>
<br />
After implementing the shading algorithm, the visual looks pretty promising. But when I rotate the light source or move the view direction I notice the terminator could sometimes becomes a sharp boundary between lit and shadow areas:

<img src="{{ site.url }}/images/specular1.gif" width="400" height="400" style="display:block; margin:auto;">
<figcaption style="text-align: center;">Artifact of the specular shader.
<br />(The terminator line gets unnatural at some angles.)</figcaption>
<br />
<!-- ```c -->
{% highlight c linenos=table %}
float3 specularReflection;
if (dot(normalDirection, lightDirection) < 0.0){
  // light source on the wrong side.
  specularReflection = float3(0.0, 0.0, 0.0); // no specular reflection.
}
else {
  // light source on the right side
  specularReflection =
    attenuation *
    _LightColor0.rgb *
    _SpecColor.rgb *
    pow(max(0.0, dot(reflect(-lightDirection, normalDirection), viewDirection)),
    _Shininess);
}

return float4(
  ambientLighting +
  diffuseReflection +
  specularReflection,
  1.0);
{% endhighlight %}
<!-- ``` -->

The specular is calculated by the dot product of the reflect vector and the view direction vector. However, this angle can be larger than 90 degrees. According to how dot product is calculated,

<img src="https://upload.wikimedia.org/math/3/e/5/3e530da12e51ca0056ed3ef061b79312.png" width="200" height="200" style="display:block; margin:auto;">
<figcaption style="text-align: center;"></figcaption>
<img src="http://mathworld.wolfram.com/images/eps-gif/Cos_600.gif" width="300" height="300" style="display:block; margin:auto;">
<figcaption style="text-align: center;"></figcaption>
When the angle between view direction and lighting direction is **larger than 90 degrees**, the cosine of it will be negative, and the dot product is also going to be negative.

Color is in the type of float4/half4/fixed4 in Cg and can not has negative elements. The negative values are simply treated as pitch black and cause this "hard terminator" artifact.

One of the ways to fix it is to multiply the final color result with the same dot product result again.

<!-- ```c -->
{% highlight c linenos=table %}
specularReflection = attenuation * _LightColor0.rgb * _SpecColor.rgb *
  pow(max(0.0, dot(reflect(-lightDirection, normalDirection), viewDirection)),
  _Shininess);
  * dot(lightDirection, normalDirection); // fixed!
{% endhighlight %}
<!-- ``` -->

Since the length of direction vectors is always 1, so this can get rid off the negative values. And this is the optimized result:

<img src="{{ site.url }}/images/specular2.gif" width="400" height="400" style="display:block; margin:auto;">
<figcaption style="text-align: center;"></figcaption>

Now the terminator is smooth! :D






<!-- The reflect vector is calculated using the Cg function -reflect()-. This function is the implement of the function  -->
