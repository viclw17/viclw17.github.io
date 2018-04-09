---
title: Recreate Phong Shading Model In Unity
date: 2016-04-19
tags:
- [Unity Shader]
- [Unity]
- [Shader]
---
![]({{ site.url }}/images/phong.png)
<!-- <img src="{{ site.url }}/images/phong.png" width="400" height="400" style="display:block; margin:auto;"> -->
<!-- <figcaption style="text-align: center;">Final work w/ transparency and faint tint and BUNNY!</figcaption> -->
<br />
[Phong shading model](https://en.wikipedia.org/wiki/Phong_reflection_model) is one of the most classic realistic shading models. It captures the basic features of a lit object using a very concise and straightforward theory. It matches people's observation in daily life, and also easy to describe using math. Therefore, Phong shading model is always used as the base for other artistic shading effects to be built on.

<img src="https://upload.wikimedia.org/wikipedia/commons/6/6b/Phong_components_version_4.png" width="600" height="400" style="display:block; margin:auto;">
<figcaption style="text-align: center;">
Visual illustration of the Phong equation.
</figcaption>

The theory is relatively simple. The final illumination of an object is **a sum of ambient lighting and all the diffuse lighting & specular  that caused by all the light sources**. Ambient lighting and diffuse lighting are usually applied globally, and specular is for object that is not matte.

<br>
 <img src=" https://upload.wikimedia.org/math/c/8/d/c8d48a83ae9c0a55afcf183b9b1ba585.png" width="400" height="400" style="display:block; margin:auto;">
<br>

* **ks**, is specular reflection constant, the ratio of reflection of the specular term of incoming light,
* **kd**, is diffuse reflection constant, the ratio of reflection of the diffuse term of incoming light (Lambertian reflectance),
* **ka**, is ambient reflection constant, the ratio of reflection of the ambient term present in all points in the scene rendered,
* **α**, is shininess constant, which is larger for surfaces that are smoother and more mirror-like. When this constant is large the specular highlight is small.
* **Lm**, is the direction vector from the point on the surface toward each light source,
* **N**, is the normal at this point on the surface,
* **Rm**, is the direction of the reflected ray of light at this point on the surface,
* **V**, is the direction pointing towards the viewer (such as a virtual camera).

**Rm** which is the reflected ray is essential for the specular effect. It is calculated with incoming ray vector, surface normal vector, and view direction vector using dot product.

<br>
 <img src=" https://upload.wikimedia.org/math/7/3/4/7341beb3c269e92781c5013a8fb8d693.png" width="200" height="200" style="display:block; margin:auto;">
<br>

<img src="https://upload.wikimedia.org/wikipedia/commons/0/01/Blinn_Vectors.svg" width="300" height="300" style="display:block; margin:auto;">
<figcaption style="text-align: center;">
Vectors for calculating Phong (or Blinn–Phong) shading.
</figcaption>

Here is the shader script for Phong shading using Cg:
<!-- ```c -->
{% highlight c linenos %}
Pass{
  Tags{ "LightMode" = "ForwardBase" }
  // pass for ambient light and first light source

  CGPROGRAM

  #pragma vertex vert  
  #pragma fragment frag

  #include "UnityCG.cginc"
  uniform float4 _LightColor0;
  // color of light source (from "Lighting.cginc")

  // User-specified properties
  uniform float4 _Color;
  uniform float4 _SpecColor;
  uniform float _Shininess;

  struct vertexInput {
    float4 vertex : POSITION;
    float3 normal : NORMAL;
  };

  struct vertexOutput {
    float4 pos : SV_POSITION;
    float4 posWorld : TEXCOORD0;
    float3 normalDir : TEXCOORD1;
  };

  vertexOutput vert(vertexInput input)
  {
    vertexOutput output;

    float4x4 modelMatrix = _Object2World;
    float4x4 modelMatrixInverse = _World2Object;

    output.posWorld =
      mul(modelMatrix, input.vertex);
    output.normalDir =
      normalize(mul(float4(input.normal, 0.0), modelMatrixInverse).xyz);
    output.pos =
      mul(UNITY_MATRIX_MVP, input.vertex);
    return output;
  }

  float4 frag(vertexOutput input) : COLOR
  {
    float3 normalDirection = normalize(input.normalDir);
    float3 viewDirection = normalize(_WorldSpaceCameraPos - input.posWorld.xyz);
    float3 lightDirection;
    float attenuation;

    if (0.0 == _WorldSpaceLightPos0.w)
    // directional light?
    {
      attenuation = 1.0; // no attenuation
      lightDirection = normalize(_WorldSpaceLightPos0.xyz);
    }
    else
    // point or spot light
    {
      float3 vertexToLightSource = _WorldSpaceLightPos0.xyz - input.posWorld.xyz;
      float distance = length(vertexToLightSource);
      attenuation = 1.0 / distance; // linear attenuation
      lightDirection = normalize(vertexToLightSource);
    }

    float3 ambientLighting =
      UNITY_LIGHTMODEL_AMBIENT.rgb * _Color.rgb;

    float3 diffuseReflection =
      attenuation * _LightColor0.rgb * _Color.rgb
      * max(0.0, dot(normalDirection, lightDirection));

    float3 specularReflection;
    if (dot(normalDirection, lightDirection) < 0.0)
    // light source on the wrong side?
    {
      specularReflection = float3(0.0, 0.0, 0.0); // no specular reflection
    }
    else
    // light source on the right side
    {
      specularReflection =
        attenuation *
        _LightColor0.rgb *
        _SpecColor.rgb *
        pow(max(0.0, dot(reflect(-lightDirection, normalDirection),viewDirection)),
        _Shininess)
        // terminator optimization
        * dot(lightDirection, normalDirection);
    }

    float4 final = float4(
    ambientLighting   
    +
    diffuseReflection
    +
    specularReflection
    , 1.0);

    return final;
  }
  ENDCG
} //END PASS
{% endhighlight %}
<!-- ``` -->
If there are more than one light source, we can add one more pass for the additional lighting effects:
<!-- ```c -->
{% highlight c linenos %}
Pass{
    Tags{ "LightMode" = "ForwardAdd" }
    // pass for additional light sources
    Blend One One
    // additive blending

    //...
    // same with the first pass

    float4 final = float4(
    diffuseReflection
    +
    specularReflection
    , 1.0);
    // no need for ambient lighting any more!

    //...
    // same with the first pass

} //END PASS
{% endhighlight %}
<!-- ``` -->

This is the testing effect:

<img src="{{ site.url }}/images/Phong/ambient.PNG" width="400" height="400" style="display:block; margin:auto;">
<figcaption style="text-align: center;">
Ambient only.
</figcaption>
<br />
<img src="{{ site.url }}/images/Phong/diffuse.PNG" width="400" height="400" style="display:block; margin:auto;">
<figcaption style="text-align: center;">
Diffuse only.
</figcaption>
<br />
<img src="{{ site.url }}/images/Phong/specular.PNG" width="400" height="400" style="display:block; margin:auto;">
<figcaption style="text-align: center;">
Specular only.
</figcaption>
<br />
<img src="{{ site.url }}/images/Phong/phong.PNG" width="400" height="400" style="display:block; margin:auto;">
<figcaption style="text-align: center;">
Final.
</figcaption>
<br />
_However, **if statement** is not optimal in shader. This is because shader is running on GPU and GPU has a highly parallel structure. I will document the optimization for the shader in future post._

V
