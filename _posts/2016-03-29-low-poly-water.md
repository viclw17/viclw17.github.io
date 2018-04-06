---
title: Low Poly Water
date: 2016-03-29
tags:
- [Unity]
- [Unity Script]
- [VFX]
---
<img src="{{ site.url }}/images/low_poly_water1.gif" width="400" height="400" style="display:block; margin:auto;">
<figcaption style="text-align: center;">Low poly water mesh with unity standard specular material.</figcaption>
<br />
My first water shader exploration in unity started with trying to **make the water surface waving**. This required mesh manipulation through shader or scripts. By playing around it, I got to know more about what other effects I can try, and above is a little side-achievement -- a nice low poly water effect.

To change the shape of a plane mesh, my first idea was to play some tricks in vertex shader -- how about **moving vertexes along their corresponding normal directions and changing the move distance randomly according to a displacement texture** (e.g. a noise texture)? Here is the code of my vertex shader:

<!-- ```c -->
{% highlight c linenos %}
v2f vert(appdata_full v) {
  v2f o;
  o.uv_DispTex = TRANSFORM_TEX(v.texcoord, _DispTex);
  o.uv_DispTex += float2(_Time.y, _Time.y) * _WavingSpeed;
  v.vertex.xyz += v.normal * tex2Dlod(_DispTex, float4(o.uv_DispTex.xy, 0, 0))
                  * _Displacement;
  o.pos = mul(UNITY_MATRIX_MVP, v.vertex);
  return o;
}
<!-- ``` -->
{% endhighlight %}

This is going to implement deformation on GPU level and the effect will come with the material assigned. And then this is the effect: (For debugging purpose I visualized the vertex normals):

<img src="{{ site.url }}/images/low_poly_fig0.gif" width="400" height="400" style="display:block; margin:auto;">
<figcaption style="text-align: center;">Low poly water mesh assigned with the vertex shader above. (Green lines are vertex normals)</figcaption>

IT IS something I want! However, as the gif shows, the drawing of the mesh is affected by the shader but the normals are not modified, which makes it impossible to achieve a beautiful low ploy flat shading style (e.g. image below). (Although the shader I wrote was unlit, since the normals are not even moved or rotated, it won't make any difference with lighting model implemented.)

<img src="{{ site.url }}/images/flat_shading.png" width="400" height="400" style="display:block; margin:auto;">
<figcaption style="text-align: center;">W/ and w/o flat shading.</figcaption>

Then I decided to go for scripting approach. This is my script to move the vertex and randomize the movement using Sine and Perlin Noise functions. This is my script:

<!-- ```c -->
{% highlight c linenos %}
public class WaterPlane : MonoBehaviour
{
  public float scale = 1.0f;
  public float sinSpeedX = 1.0f;
  public float sinSpeedZ = 1.0f;
  public float perlinSpeedX = 1.0f;
  public float perlinSpeedZ = 1.0f;
  private Vector3[] baseVertices;
  public bool recalculateNormals = true;

  public bool isSin = false;
  public bool isPerlin = true;

  Mesh mesh;
  Vector3[] vertices;

  void Start () {
    mesh = GetComponent<MeshFilter>().mesh;

    if (baseVertices == null)
        baseVertices = mesh.vertices;

    vertices = new Vector3[baseVertices.Length];
  }

  void Update () {
      for (int i = 0; i < vertices.Length; i++)
      {
        Vector3 vertex = baseVertices[i];

        if (isSin == true && isPerlin == false) {
          vertex.y += Mathf.Sin(vertex.x + Time.time * sinSpeedX) *
                      Mathf.Sin(vertex.z + Time.time * sinSpeedZ) * scale;
        }
        if (isPerlin == true && isSin == false) {
          vertex.y += Mathf.PerlinNoise(vertex.x + Time.time * perlinSpeedX,
                                        vertex.z + Time.time * perlinSpeedZ)* scale;
        }
        if (isPerlin == true && isSin == true)
        {
          vertex.y += Mathf.PerlinNoise(vertex.x + Time.time * perlinSpeedX,
                                        vertex.z + Time.time * perlinSpeedZ) *
                      Mathf.Sin(vertex.x + Time.time * sinSpeedX) *
                      Mathf.Sin(vertex.z + Time.time * sinSpeedZ) * scale;
        }

        vertices[i] = vertex;
      }

      mesh.MarkDynamic();
      mesh.vertices = vertices;
      mesh.RecalculateBounds();

      if (recalculateNormals)
    		mesh.RecalculateNormals ();

    }
  }
}
<!-- ``` -->
{% endhighlight %}

Since it was only on CPU level, we can apply material with any shader we want, which is great. The effect was like this:

<img src="{{ site.url }}/images/low_poly_fig2.gif" width="400" height="400" style="display:block; margin:auto;">
<figcaption style="text-align: center;">Low poly water mesh assigned with the script above. (Green lines are vertex normals)</figcaption>
<!-- <img src="{{ site.url }}/images/low_poly_fig3.gif" width="400" height="400" style="display:block; margin:auto;">
<figcaption style="text-align: center;">Low poly water mesh assigned with the script above. (Green lines are vertex normals)</figcaption> -->

Note that the line **mesh.RecalculateNormals ();** is very important. If we remove it, the vertexes are moved but the normals are not updated and they will keep pointing upward, so the shading will look like smooth. And we will lost the nice flat shading:

<img src="{{ site.url }}/images/low_poly_fig1.gif" width="400" height="400" style="display:block; margin:auto;">
<figcaption style="text-align: center;">W/o mesh.RecalculateNormals (); (Green lines are vertex normals, )</figcaption>

However, this effect will be eventually useful when I want to implement the real water surface effect. I will show that in Water Shader Exploration Part-2.

Now I can simply use the powerful build-in Standard shader and tweak the smoothness and the transparency to achieve a nice low poly water effect. :D

<img src="{{ site.url }}/images/low_poly_water2.gif" width="400" height="400" style="display:block; margin:auto;">
<!-- <figcaption style="text-align: center;">W/o mesh.RecalculateNormals (); (Green lines are vertex normals, )</figcaption> -->
