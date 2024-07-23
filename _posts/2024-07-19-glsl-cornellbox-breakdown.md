---
title: "GLSL Cornell Box Project Breakdown - Part 1"
type: article
layout: post

---
This post first listed the common implement choices for path tracing with resources. But the main goal is aiming to break down yumcyawiz's project [glsl330-cornellbox](https://github.com/yumcyaWiz/glsl330-cornellbox). This project is a brilliant demo of path tracing implemented using GLSL shader code, but is presented interactively as a standalone executable using basic OpenGL and Imgui in C++. It is a great example for reviewing OpenGL workflow as well as path tracing implementation.

* This will become a table of contents (this text will be scrapped).
{:toc}

---

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
The project launches a `GLFWwindow` from `main()`, in `main.h` 

In the **main app while-loop** of the GLFW window, it creates an **Imgui frame** and draws all the UI elements. It then calls `renderer -> render()` function which is doing the most heavy-lifting.

The `Renderer::render()` in `renderer.h` is initializing all the **C++ objects**, as well as setting up all the **OpenGL drawing objects and shaders objects**. Then it draws the result of **GLSL path tracing** to the screen quad, based on several switch cases.

`Rectangle` in `rectangle.h` is setting up the only **OpenGL geometry objects** - a quad; and functions to draw and destroy it.

`Scene` in `scene.h` contains instructions of setting up the cornell box scene, from geometry properties to their transform and materials parameters. 

To do that, it reserves a memory blocks `SceneBlock`, `Primitive`, `Material` and `Light` to cache all the scene description data - most them are `int`, `float` and `glm::vec3`. 

Note that it is only packing up the data with an organized interface, and eventually the data is sent to shaders at `Renderer` to put into use.

Camera in camera.h, is similar. It reserves a memory block `CameraBlock`, and defines function for the math operations to move and orbit it.

Finally, the rest of the path tracing is done purely in fragment shaders, which I will cover in part 2.



# External packages:
- Imgui
- GLFW
- GLAD
- GLM
- GLSL-Shader-Includes

<img src="{{ site.url }}/images\2024-07-19-glsl-cornellbox-breakdown\image.png" width="400" style="display:block; margin:auto;">

<!-- ## OpenGL library on Windows -->
<!-- If you're on Windows the OpenGL library opengl32.lib comes with the Microsoft SDK, which is installed by default when you install Visual Studio. -->

<!-- GLFW is a library, written in C, specifically targeted at OpenGL. GLFW gives us the bare necessities required for rendering goodies to the screen. It allows us to create an OpenGL context, define window parameters, and handle user input, which is plenty enough for our purposes. -->

<!-- [GLM](https://learnopengl.com/Getting-started/Transformations) -->

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
# renderer.h
Define the class of `Renderer`. It holds references to:

- sample number
- *GlobalBlock object*
- Camera object
- Scene object
- Rectangle object
- id of GL texture objects: `accumTexture`, `stateTexture`, `accumFBO` (Frame Buffer Object)
- id of GL UBO (Uniform Buffer Object)
- Shader objects (pt_shader)
- enum variable, RenderMode (Render, Normal, Depth, Albedo, UV), for visualization
- enum variable, Integrator (right now only focus on PT)
- enum variable, SceneType 

Note that the renderer simply only render 1 quad (prepared by Rectangle class) to the screen, and all path tracing render is done in fragment shader.

## Renderer()

On construction, the constructor will firstly initialize all the objects above. It will also setup/generate those **GL objects** and track their ids with those id variables.

Texture objects:

```c
// setup accumulate texture
glGenTextures(1, &accumTexture);
glBindTexture(GL_TEXTURE_2D, accumTexture);
glTexImage2D(GL_TEXTURE_2D, 0, GL_RGB32F, width, height, 0, GL_RGB, GL_FLOAT, 0);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_NEAREST);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_NEAREST);
glBindTexture(GL_TEXTURE_2D, 0);
```

Here it also setup a special texture:
```c
// setup RNG state texture
glGenTextures(1, &stateTexture);
glBindTexture(GL_TEXTURE_2D, stateTexture);
// reserve a container *seed* of size of the full pixel amount
std::vector<uint32_t> seed(width * height);
std::random_device rnd_dev;
std::mt19937 mt(rnd_dev());
std::uniform_int_distribution<uint32_t> dist(1, std::numeric_limits<uint32_t>::max());
// fill each element of *seed* with a random number
for (unsigned int i = 0; i < seed.size(); ++i) {
  seed[i] = dist(mt);
}
glTexImage2D(GL_TEXTURE_2D, 0, GL_R32UI, width, height, 0, GL_RED_INTEGER, GL_UNSIGNED_INT, seed.data());
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_NEAREST);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_NEAREST);
glBindTexture(GL_TEXTURE_2D, 0);
```

Frame buffer obj:
```c
// setup accumulate FBO
glGenFramebuffers(1, &accumFBO);
glBindFramebuffer(GL_FRAMEBUFFER, accumFBO);
glFramebufferTexture2D(GL_FRAMEBUFFER, GL_COLOR_ATTACHMENT0, GL_TEXTURE_2D, accumTexture, 0);
glFramebufferTexture2D(GL_FRAMEBUFFER, GL_COLOR_ATTACHMENT1, GL_TEXTURE_2D, stateTexture, 0);
GLuint attachments[2] = {GL_COLOR_ATTACHMENT0, GL_COLOR_ATTACHMENT1};
glDrawBuffers(2, attachments);
glBindFramebuffer(GL_FRAMEBUFFER, 0);
```

Setup UBOs - globalUBO, cameraUBO, sceneUBO:
```c
// setup UBO
glGenBuffers(1, &globalUBO);
glBindBuffer(GL_UNIFORM_BUFFER, globalUBO);
glBufferData(GL_UNIFORM_BUFFER, sizeof(GlobalBlock), &global, GL_DYNAMIC_DRAW);
glBindBuffer(GL_UNIFORM_BUFFER, 0);
// ...
glBindBufferBase(GL_UNIFORM_BUFFER, 0, globalUBO);
// ...
```

Next is to send those uniforms and GL objects into shader programs (like pt_shader):
```c
// set uniforms
pt_shader.setUniformTexture("accumTexture", accumTexture, 0);
pt_shader.setUniformTexture("stateTexture", stateTexture, 1);
pt_shader.setUBO("GlobalBlock", 0);
pt_shader.setUBO("CameraBlock", 1);
pt_shader.setUBO("SceneBlock", 2);
// ...
output_shader.setUniformTexture("accumTexture", accumTexture, 0);
```

`destroy()` function is calling those GL functions to delete object:

```c
glDeleteTextures(1, &accumTexture);
glDeleteFramebuffers(1, &accumFBO);
glDeleteBuffers(1, &globalUBO);

pt_shader.destroy();

rectangle.destroy();
```
 
Few other member functions:
- glm::vec3 getCameraPosition();
- void setFOV(float fov);
- void moveCamera(const glm::vec3& v)
- void orbitCamera(float dTheta, float dPhi)

## Render::render()
`render()` function is firstly defining glViewport(). Then it switches by **RenderMode** enum - here we focus on mode **Render**. Then we bind FBO by `glBindFramebuffer(GL_FRAMEBUFFER, accumFBO)`, which will get the drawing ready. 

Then it switches by **Integrator** enum - here we focus on PT: `rectangle.draw(pt_shader)`.

This will draw one pass of the path tracing result, which equal to pick one sample. Then it increments the samples count: `samples++`

Remember to unbound FBO by `glBindFramebuffer(GL_FRAMEBUFFER, 0)`

Because we are doing progressive rendering here, the accumFBO - just as its name - is accumulating all sampled path tracing frams and now it will divide the sum by sample count to draw to the output shader:

```c
// output
output_shader.setUniform("samplesInv", 1.0f / samples);
rectangle.draw(output_shader);
```


# shader.h
It defined the infrastructure class `Shader` to configure OpenGL shader objects.

## Shader()
Constructor is taking in **shader file paths**, then **compile** shaders and **link** the shaders.

Compile shader:

```c
void compileShader() {
  // compile vertex shader
  vertex_shader = glCreateShader(GL_VERTEX_SHADER);
  vertex_shader_source = Shadinclude::load(vertex_shader_filepath);
  const char* vertex_shader_source_c_str = vertex_shader_source.c_str();
  glShaderSource(vertex_shader, 1, &vertex_shader_source_c_str, nullptr);
  glCompileShader(vertex_shader);

  // handle compilation error
  GLint success = 0;
  glGetShaderiv(vertex_shader, GL_COMPILE_STATUS, &success);
  if (success == GL_FALSE) {
    //...
    glDeleteShader(vertex_shader);
    return;
  }

  // compile fragment shader
  fragment_shader = glCreateShader(GL_FRAGMENT_SHADER);
  fragment_shader_source = Shadinclude::load(fragment_shader_filepath);
  const char* fragment_shader_source_c_str = fragment_shader_source.c_str();
  glShaderSource(fragment_shader, 1, &fragment_shader_source_c_str, nullptr);
  glCompileShader(fragment_shader);

  // handle compilation error
}
```

Link shader:

```c
  void linkShader() {
  // Link Shader Program
  program = glCreateProgram();
  glAttachShader(program, vertex_shader);
  glAttachShader(program, fragment_shader);
  glLinkProgram(program);
  glDetachShader(program, vertex_shader);
  glDetachShader(program, fragment_shader);

  // handle link error
  int success = 0;
  glGetProgramiv(program, GL_LINK_STATUS, &success);
  if (success == GL_FALSE) {
    // ...
    glDeleteProgram(program);
    return;
  }
}
```

Helper functions to active/deactive shaders:
```c
void activate() const { glUseProgram(program); }
void deactivate() const { glUseProgram(0); }
```

Then it prepares more helper functions to set **uniforms** values in the shader. This is a typical practice and those functions are defined for **every single data type**:

- `glUniform1i`
- `glUniform1ui`
- `glUniform1f`
- `glUniform2fv`
- `glUniform2uiv`
- `glUniform3fv`

```c
void setUniform(const std::string& uniform_name, GLint value) const {
  activate();
  const GLint location = glGetUniformLocation(program, uniform_name.c_str());
  glUniform1i(location, value);
  deactivate();
}

// ...

void setUniformTexture(const std::string& uniform_name, GLuint texture, GLuint texture_unit_number) const {
  activate();
  const GLint location = glGetUniformLocation(program, uniform_name.c_str());
  glUniform1i(location, texture_unit_number);
  glActiveTexture(GL_TEXTURE0 + texture_unit_number);
  glBindTexture(GL_TEXTURE_2D, texture);
  deactivate();
}

void setUBO(const std::string& block_name, GLuint binding_number) const {
  const GLuint index = glGetUniformBlockIndex(program, block_name.c_str());
  glUniformBlockBinding(program, index, binding_number);
}
```

# rectangle.h
Define the `Rectangle` class to prepare to draw the screen quad.

## Rectangle()
```c
class Rectangle {
 private:
  GLuint VBO;
  GLuint EBO;
  GLuint VAO;

 public:
  Rectangle() {
    // setup VAO
    glGenVertexArrays(1, &VAO);
    glBindVertexArray(VAO);

    // setup VBO;
    // positions and texture coords
    GLfloat vertices[] = {-1.0f, -1.0f, 0.0f, 0.0f, 0.0f, 1.0f, -1.0f,
                          0.0f,  1.0f,  0.0f, 1.0f, 1.0f, 0.0f, 1.0f,
                          1.0f,  -1.0f, 1.0f, 0.0f, 0.0f, 1.0f};
    glGenBuffers(1, &VBO);
    glBindBuffer(GL_ARRAY_BUFFER, VBO);
    glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);

    // setup EBO;
    GLuint indices[] = {0, 1, 2, 2, 3, 0};
    glGenBuffers(1, &EBO);
    glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, EBO);
    glBufferData(GL_ELEMENT_ARRAY_BUFFER, sizeof(indices), indices,
                 GL_STATIC_DRAW);

    // position attribute
    glEnableVertexAttribArray(0);
    glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 5 * sizeof(GLfloat),
                          (GLvoid*)0);
    glEnableVertexAttribArray(1);
    glVertexAttribPointer(1, 2, GL_FLOAT, GL_FALSE, 5 * sizeof(GLfloat),
                          (GLvoid*)(3 * sizeof(float)));

    // unbind VAO, VBO, EBO
    glBindVertexArray(0);
    glBindBuffer(GL_ARRAY_BUFFER, 0);
    glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, 0);
  }
  ```

  ```c

  void destroy() {
    glDeleteBuffers(1, &VBO);
    glDeleteBuffers(1, &EBO);
    glDeleteVertexArrays(1, &VAO);
  }

  void draw(const Shader& shader) const {
    shader.activate();
    glBindVertexArray(VAO);
    glDrawElements(GL_TRIANGLES, 6, GL_UNSIGNED_INT, 0);
    glBindVertexArray(0);
    shader.deactivate();
  }
};
```

# scene.h
`Scene` class contains definition of the Cornell Box.

```c
struct alignas(16) Primitive {
  int id;                                 // 4
  int type;                               // 8
  alignas(16) glm::vec3 center;           // 24
  float radius;                           // 28
  alignas(16) glm::vec3 leftCornerPoint;  // 44
  alignas(16) glm::vec3 up;               // 60
  alignas(16) glm::vec3 right;            // 76
  int material_id;                        // 80
};

struct alignas(16) Material {
  int brdf_type;             // 4
  alignas(16) glm::vec3 kd;  // 20
  alignas(16) glm::vec3 le;  // 36
};

struct alignas(16) Light {
  int primID;
  alignas(16) glm::vec3 le;
};

struct alignas(16) SceneBlock {
  int n_materials;
  int n_primitives;
  int n_lights;
  Material materials[100];
  Primitive primitives[100];
  Light lights[100];
};

enum class SceneType {
  Original,
  Sphere,
  Indirect,
};
```

```c
class Scene {
 private:
  void setupCornellBoxOriginal() {  }


  void init() {}

  void clear() {
    n_primitives = 0;
    n_materials = 0;
  }

 public:
  int n_primitives;
  int n_materials;
  SceneBlock block;

  void addPrimitive(const Primitive& primitive) {}

  void addMaterial(const Material& material) {}

  static Primitive createSphere(const glm::vec3& center, float radius) {}

  static Primitive createPlane(const glm::vec3& leftCornerPoint,
                               const glm::vec3& right, const glm::vec3& up) {}

  static Material createDiffuse(const glm::vec3& kd) {}

  static Material createMirror(const glm::vec3& kd) {}

  static Material createGlass(const glm::vec3& kd) {}

  static Material createLight(const glm::vec3& le) {}

  Scene() : n_primitives(0), n_materials(0) {}

  void setScene(const SceneType& scene_type) {}
};
```

# camera.h
```c
struct alignas(16) CameraBlock {
  alignas(16) glm::vec3 camPos;
  alignas(16) glm::vec3 camForward;
  alignas(16) glm::vec3 camRight;
  alignas(16) glm::vec3 camUp;
  float a;

  CameraBlock(const glm::vec3& camPos, const glm::vec3& camForward,
              const glm::vec3& camRight, const glm::vec3& camUp)
      : camPos(camPos),
        camForward(camForward),
        camRight(camRight),
        camUp(camUp) {}
};

class Camera {
 public:
  CameraBlock params;
  float fov;

 private:
  glm::vec3 lookat;

 public:
  Camera()
      : params({278, 273, -900}, {0, 0, 1}, {-1, 0, 0}, {0, 1, 0}),
        fov(0.25 * PI),
        lookat({278, 273, 279.6}) {
    setFOV(fov);
  }

  void setFOV(float fov) {}

  void move(const glm::vec3& v) {}

  void orbit(float dTheta, float dPhi) {}
};
```


TBC




