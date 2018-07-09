---
title: GLSL Practice With Shadertoy
date: 2018-06-12
tags:
- GLSL
- Shader
- Shadertoy
- UV
---
<!---
Featured image. width set to 640 to align with shadertoy
-->
<img src="{{ site.url }}/images/glsl-jupiter.jpg" width="640"  style="display:block; margin:auto;">
<!-- <figcaption style="text-align: center;">First PBR rendering test, looking neat. </figcaption> -->
<br />
Getting started to use [Shadertoy](https://www.shadertoy.com/) to learn and practice GLSL. Here are the first few examples I've been playing around. Also got the shaders embedded in my blog page.
Here I documented some of my exploration about the website and some best-practice.
(_I also use [KodeLife](https://hexler.net/software/kodelife/) to work on my shader offline. It is an amazing live shader programming tool and I will document the basic usages of it in next blog post._)

# Shadertoy First Try
## Watercolor Blending
Made a simple shader iterating a combination of sine and cosine functions with UV coordinate to achieve a watercolor blending effect. Shader is very simple but the final look is very organic.

_* Use mouse click and drag to interact with the motion._
<br>
<iframe width="640" height="360" frameborder="0" src="https://www.shadertoy.com/embed/lsyfWD?gui=true&t=10&paused=false&muted=false" allowfullscreen style="display:block; margin:auto;"></iframe>
<br>
{% highlight c linenos=table %}
void mainImage( out vec4 fragColor, in vec2 fragCoord )
{   
    float speed = .1;
    float scale = 0.002;
    vec2 p = fragCoord * scale;   
    for(int i=1; i<10; i++){
        p.x+=0.3/float(i)*sin(float(i)*3.*p.y+iTime*speed)+iMouse.x/1000.;
        p.y+=0.3/float(i)*cos(float(i)*3.*p.x+iTime*speed)+iMouse.y/1000.;
    }
    float r=cos(p.x+p.y+1.)*.5+.5;
    float g=sin(p.x+p.y+1.)*.5+.5;
    float b=(sin(p.x+p.y)+cos(p.x+p.y))*.5+.5;
    vec3 color = vec3(r,g,b);
    fragColor = vec4(color,1);
}
{% endhighlight %}

## Jupiter
I then take the watercolor blending shader to the next level - mapping the final color output onto a UV sphere and added scrolling and stretching motion. The final looking is just like a Jupiter planet.
<!-- I decide to keep pushing this Jupiter shader and add lighting model and fresnel shading etc. in the future to make it more close to the real look of the planet. -->
<br>
<iframe width="640" height="360" frameborder="0" src="https://www.shadertoy.com/embed/MdyfWw?gui=true&t=10&paused=false&muted=false" allowfullscreen style="display:block; margin:auto;"></iframe>
<br>
Later I added lighting model and background to make it more realistic. It is amazing to see how much a simple uv distortion can achieve.
<br>
<iframe width="640" height="360" frameborder="0" src="https://www.shadertoy.com/embed/XsVBWG?gui=true&t=10&paused=false&muted=false" allowfullscreen style="display:block; margin:auto;"></iframe>
<br>

# Shadertoy Basics
[Shadertoy](https://www.shadertoy.com/) is a online community and real-time browser tool for creating and sharing GLSL shaders. It provides the **boilerplate code** for GLSL shader programming so that we can just jump onto the most creative part of graphic programming. We can also get access to thousands of shaders written by Computer graphics professionals, academics and enthusiasts, and learn by tweaking their code.

## Nice reference sites
- [Fabrice's SHADERTOY – UNOFFICIAL](https://shadertoyunofficial.wordpress.com/)
- [Fabrice's SHADERTOY – UNOFFICIAL - Usual tricks in Shadertoy / GLSL](https://shadertoyunofficial.wordpress.com/2016/07/21/usual-tricks-in-shadertoyglsl/)
- [Official How-To](https://www.shadertoy.com/howto)

## Boilerplacte Code
```c
// Shader Inputs, uniforms
uniform vec3      iResolution;           // viewport resolution (in pixels)
uniform float     iTime;                 // shader playback time (in seconds)
uniform float     iTimeDelta;            // render time (in seconds)
uniform int       iFrame;                // shader playback frame
uniform float     iChannelTime[4];       // channel playback time (in seconds)
uniform vec3      iChannelResolution[4]; // channel resolution (in pixels)
uniform vec4      iMouse;                // mouse pixel coords. xy: current (if MLB down), zw: click
uniform samplerXX iChannel0..3;          // input channel. XX = 2D/Cube
uniform vec4      iDate;                 // (year, month, day, time in seconds)

// Main funtion
void mainImage( out vec4 fragColor, in vec2 fragCoord )
{
    // Normalized pixel coordinates (from 0 to 1)
    vec2 uv = fragCoord/iResolution.xy;

    // Time varying pixel color
    vec3 col = 0.5 + 0.5*cos(iTime+uv.xyx+vec3(0,2,4));

    // Output to screen
    fragColor = vec4(col,1.0);
}
```
## Tutorial Notes
### mainImage()
Image shaders implement the mainImage() function in order to generate the procedural images by computing a color for each pixel. This function is expected to be called once per pixel, and it is responsability of the host application to provide the right inputs to it and get the output color from it and assign it to the screen pixel.

### fragCoord
where fragCoord contains the pixel coordinates for which the shader needs to compute a color. The coordinates are in pixel units, ranging from 0.5 to resolution-0.5, over the rendering surface, where the resolution is passed to the shader through the iResolution uniform.

### fragColor
The resulting color is gathered in fragColor as a four component vector, the last of which is ignored by the client. The result is gathered as an "out" variable in prevision of future addition of multiple render targets.

# Code Compatibility
## Kodelife
Make sure creating new project from Shadertoy Template. It comes with these kind of code. Simply copy-paste Shadertoy code under all of them to run the shader in KodeLife.
```c
#version 150

in VertexData
{
    vec4 v_position;
    vec3 v_normal;
    vec2 v_texcoord;
} inData;

out vec4 fragColor;

////////////////////////////////////////////////////////////////////////////////
////////////////////////////////////////////////////////////////////////////////

uniform vec2 iResolution;
uniform float iTime;
uniform float iTimeDelta;
uniform int iFrame;
uniform vec4 iMouse;
uniform sampler2D iChannel0;
uniform sampler2D iChannel1;
uniform sampler2D iChannel2;
uniform sampler2D iChannel3;
uniform vec4 iDate;
uniform float iSampleRate;

void mainImage(out vec4, in vec2);
void main(void) { mainImage(fragColor,inData.v_texcoord * iResolution.xy); }

////////////////////////////////////////////////////////////////////////////////
////////////////////////////////////////////////////////////////////////////////
```
_To-Do: GLSL Sandbox, Atom Editor Compatibility._
