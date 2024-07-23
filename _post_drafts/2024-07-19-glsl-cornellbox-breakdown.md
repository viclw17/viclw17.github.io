---
title: "GLSL Cornell Box Project Breakdown"
type: article
layout: post
image: 2024-07-20-glsl-cornellbox-breakdown/cover.jpg
---
This post first listed the common implement choices for path tracing with resources. But the main goal is aiming to break down yumcyawiz's project [glsl330-cornellbox](https://github.com/yumcyaWiz/glsl330-cornellbox). This project is a brilliant demo of path tracing implemented using GLSL shader code, but is presented interactively as a standalone executable using basic OpenGL and Imgui in C++. It is a great example for reviewing OpenGL workflow as well as path tracing implementation.

* This will become a table of contents (this text will be scrapped).
{:toc}

# Explore Path Tracing Implement Choices
During my research, as a hobbist there are many go-to ways to implement basic path tracing:

- Write in C++ from ground up, calculate pixel values and output images file(eg. PPM), a few bookmarked examples are:
  - RIOW
  - [PBRT](https://github.com/mmp/pbrt-v3)
  - [smallpt](https://www.kevinbeason.com/smallpt/)
  - [Montelight](https://github.com/Smerity/montelight-cpp)
  - [NanoRT](https://github.com/viclw17/nanort)
  - [Luminox](https://github.com/yumcyaWiz/Luminox)
  - [Aras' ToyPathTracer](https://github.com/aras-p/ToyPathTracer)

This option is mainly for off-line rendering done solely on CPU, with less interactivity. It usually takes long time - according to how optimized - for the result to converge and an image to be generated. But this is the most straightforward approach in computer graphics research as well as in the industries.

- Utilize direct GPU programming:
  - [Ray Tracing in One Weekend in CUDA](https://github.com/rogerallen/raytracinginoneweekendincuda)

This option can be challenging and requires knowledge on GPU programming libraries.

- Write in shader code like GLSL/HLSL that is running on GPU:
  - **Shadertoy**, has so many amazing path tracing demos done in 1 or few passes through fragment shader
  - written in shader code but run through local **Graphic API** like OpenGL, eg. [glsl330-cornellbox](https://github.com/yumcyaWiz/glsl330-cornellbox) or [GLSL-PathTracer](https://github.com/knightcrawler25/GLSL-PathTracer).

This option could be difficult for people are not familliar with shading languages, but with the power of GPU the path tracing scene can be interacted via user inputs (but not real-time!) and it may just take seconds for the result to converge with **progressive rendering**. With shader you can also explore modeling using SDF, and then shade the scene using path tracing.

<iframe width="100%"  height="400" src="https://www.youtube.com/embed/-dmQk2q3FTo?si=Kgzeyq-2eKlO2gQx" frameborder="0" allowfullscreen style="display:block; margin:auto;"></iframe>

---

# Breakdown
The project is built in C++ to launch a `GLFWwindow`. In the while() main app loop, it created an **Imgui frame** for all the UI, and called `renderer -> render()` function to render path tracing in the GLSL shaders. 

# External packages:
- Imgui
- GLFW
- GLAD
- GLM
- GLSL-Shader-Includes

## OpenGL library on Windows
If you're on Windows the OpenGL library opengl32.lib comes with the Microsoft SDK, which is installed by default when you install Visual Studio. Since this chapter uses the VS compiler and is on windows we add opengl32.lib to the linker settings. 

## GLFW
GLFW is a library, written in C, specifically targeted at OpenGL. GLFW gives us the bare necessities required for rendering goodies to the screen. It allows us to create an OpenGL context, define window parameters, and handle user input, which is plenty enough for our purposes.

![alt text](images/2024-07-19-glsl-cornellbox-breakdown/image.png)

## GLAD
- initialize glad

### GLM
[text](https://learnopengl.com/Getting-started/Transformations)

# main.cpp
A good starting point would be the [example_glfw_opengl3](https://github.com/ocornut/imgui/tree/master/examples/example_glfw_opengl3) of Imgui in the repository. It offers a good template I believe this project is based on.

<img src="{{ site.url }}/images\2024-07-19-glsl-cornellbox-breakdown\imgui_example.png" width="640" style="display:block; margin:auto;">

<!-- ## handleInput -->
## main()
- [Creating A Window](https://learnopengl.com/Getting-started/Creating-a-window)

```c
// init glfw

// set glfw error callback

// setup window and context
glfwWindowHint(GLFW_CONTEXT_VERSION_MAJOR, 3);
glfwWindowHint(GLFW_CONTEXT_VERSION_MINOR, 3);
glfwWindowHint(GLFW_OPENGL_PROFILE, GLFW_OPENGL_CORE_PROFILE);
glfwWindowHint(GLFW_OPENGL_FORWARD_COMPAT, GL_TRUE);  // Required on Mac

GLFWwindow* window =
    glfwCreateWindow(1000, 1000, "GLSL CornellBox", nullptr, nullptr);

// set glfw window error
if (!window) {
  std::cerr << "failed to create window" << std::endl;
  std::exit(EXIT_FAILURE);
}

glfwMakeContextCurrent(window);

// initialize glad
```
- setup imgui context
- setup Dear ImGui style
- set up renderer 

``` c
renderer = std::make_unique<Renderer>(1000, 1000);
```

### main app loop
```c
// main app loop
  while (!glfwWindowShouldClose(window)) {
    glfwPollEvents();

    // Start the Dear ImGui frame
    ImGui_ImplOpenGL3_NewFrame();
    ImGui_ImplGlfw_NewFrame();
    ImGui::NewFrame();

    ImGui::Begin("Renderer");
    {
      // def gui design and hook it with the renderer settings like:
      // renderer->resize();
      // renderer->setRenderMode();
      // renderer->setIntegrator();
      // renderer->setSceneType();
      // renderer->getSamples();
      // renderer->getCameraPosition();
      // renderer->getCameraFOV();
      // renderer->setFOV();
      // ...

    }
    ImGui::End();

    // Handle Input
    handleInput(window, io);

    // Rendering
    glClear(GL_COLOR_BUFFER_BIT);

    renderer->render();

    // ImGui Rendering
    int display_w, display_h;
    glfwGetFramebufferSize(window, &display_w, &display_h);

    glViewport(0, 0, display_w, display_h);

    ImGui::Render();

    ImGui_ImplOpenGL3_RenderDrawData(ImGui::GetDrawData());

    // gl
    glfwSwapBuffers(window);
  }
```
- shutdown Imgui
- destroy renderer object
- shutdown GL window

# rectangle.h
basically the shader output screen

class that hold data of the geo, and its opengl objects

function to draw

# shader.h
class to hold opengl shader objects

- compile shader
- link shader

functions to set uniforms for shader program

# scene.h

# camera.h
class to hold cam info

functions to set fov, move, and orbit


# renderer.h
this is the opengl renderer, which put a quad onto the screen/ app window

create rectangle object and shader objects

setup accumulate texture

setup rng state texture

setup UBO

pass the values of the uniforms into the shaders

pass the camera info into the shaders

```render``` function to call ```draw``` function on rectangle based on path tracing method option and visualization mode

# GLSL Shaders
This part is very similar to Path Tracing Workshop I've covered.

But the code here are very well refactored and are splitted into multiple files based on their goals. They form a great mind map for the path tracing system I've studied so far.

They are mainly fragment shaders for pixel value calculation. There is only one vertex shader for setting up the rectangle.

## pt.frag

TBC




