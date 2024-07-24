---
title: "Study Notes: GLSL CornellBox Breakdown - Part 1"
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

In the **main app while-loop** of the GLFW window, it creates an **Imgui frame** and draws all the UI elements. It then calls `renderer -> render()` which is doing the most heavy-lifting.

The constructor of `Renderer` class in `renderer.h` is initializing all the **C++ objects**, as well as setting up all the **OpenGL drawing objects and shaders objects**. Then it draws the result of **GLSL path tracing** to the screen quad, based on several switch cases.

`Shader` class in `shader.h` prepares infrastructure for OpenGL shader compiling and linking.

`Rectangle` in `rectangle.h` sets up the only **OpenGL geometry objects** - a quad; and functions to draw and destroy it.

`Scene` class in `scene.h` contains instructions of setting up the cornell box scene, from geometry properties to their transform and materials parameters. It reserves several memory blocks `SceneBlock`, `Primitive`, `Material` and `Light` to cache all the scene description data - mainly `int`, `float` and `glm::vec3`. Note that it is only packing up the data with an organized way, and eventually the data is sent to shaders at `Renderer` to put into use.

`Camera` class in `camera.h` reserves a memory block `CameraBlock`, and defines functions for the math operations to move and orbit it.

Finally, the rest of the path tracing is done purely in GLSL fragment shaders, which I will cover in part 2.



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
After some research, I found a good starting point would be the [example_glfw_opengl3](https://github.com/ocornut/imgui/tree/master/examples/example_glfw_opengl3) of Imgui in the repository. It offers a good template I believe this project is based on.

<img src="{{ site.url }}/images\2024-07-19-glsl-cornellbox-breakdown\imgui_example.png" width="640" style="display:block; margin:auto;">

<!-- ## handleInput -->
## main()
### Create OpenGL Window
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
- setup Imgui context
- setup ImGui style

### Create the renderer

``` c
renderer = std::make_unique<Renderer>(1000, 1000);
```

### The main app loop
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
    // def imgui design and hook it with the renderer settings like:

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

  // Render
  glClear(GL_COLOR_BUFFER_BIT);
  renderer->render();

  // ImGui Render
  int display_w, display_h;
  glfwGetFramebufferSize(window, &display_w, &display_h);
  glViewport(0, 0, display_w, display_h);
  ImGui::Render();
  ImGui_ImplOpenGL3_RenderDrawData(ImGui::GetDrawData());

  // gl
  glfwSwapBuffers(window);
}
```

after the loop:

- shutdown Imgui
- destroy renderer object
- shutdown GL window
  
# renderer.h
Define the class of `Renderer`. It holds references to:

- sample number
- *GlobalBlock memory struct*
- Camera object
- Scene object
- Rectangle object
- id of GL texture objects: `accumTexture`, `stateTexture`, `accumFBO`(Frame Buffer Object)
- id of GL UBO (Uniform Buffer Object): `globalUBO`, `cameraUBO`, `sceneUBO`
- Rectangle object
- Shader objects (pt_shader)
- enum variable, RenderMode (Render, Normal, Depth, Albedo, UV), for visualization
- enum variable, Integrator (PT)
- enum variable, SceneType (Original Cornell Box Scene)

Note that the renderer simply only render 1 quad (prepared by Rectangle class) to the screen.

## Renderer()
On construction, the constructor will firstly initialize all the objects above. It will also setup/generate those **GL objects** and track their ids with those id variables.

### Create Texture Objects

```c
// setup accumulate texture
glGenTextures(1, &accumTexture);
glBindTexture(GL_TEXTURE_2D, accumTexture);
glTexImage2D(GL_TEXTURE_2D, 0, GL_RGB32F, width, height, 0, GL_RGB, GL_FLOAT, 0);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_NEAREST);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_NEAREST);
glBindTexture(GL_TEXTURE_2D, 0);
```

Here it also setup a special texture, to save random number generator state for every single pixel:
```c
// setup RNG state texture
glGenTextures(1, &stateTexture);
glBindTexture(GL_TEXTURE_2D, stateTexture);

// reserve a container *seed* of size of the full pixel amount
std::vector<uint32_t> seed(width * height);
std::random_device rnd_dev;
std::mt19937 mt(rnd_dev());

// Produces random integer values i, uniformly distributed on the closed interval [a,b], that is, 
// distributed according to the discrete probability function
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

### Create Frame Buffer Object
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

### Create Uniform Buffer Objects
Setup UBOs - `globalUBO`, `cameraUBO`, `sceneUBO`:
```c
// setup UBO
// GlobalBlock
glGenBuffers(1, &globalUBO);
glBindBuffer(GL_UNIFORM_BUFFER, globalUBO);
glBufferData(GL_UNIFORM_BUFFER, sizeof(GlobalBlock), &global, GL_DYNAMIC_DRAW);
glBindBuffer(GL_UNIFORM_BUFFER, 0);
// CameraBlock...
// SceneBlock...

glBindBufferBase(GL_UNIFORM_BUFFER, 0, globalUBO);
// CameraBlock...
// SceneBlock...
```

### Send GL Objects to Shader Uniforms
Next is to send those GL objects into shader programs(pt_shader) via uniforms:
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
//...
glDeleteFramebuffers(1, &accumFBO);
//...
glDeleteBuffers(1, &globalUBO);

pt_shader.destroy();
//...

rectangle.destroy();
```

### Send Camera Param to Shader 
Few functions for camera object will send camera parameters into OpenGL:

- `glm::vec3 getCameraPosition()`
- `void setFOV(float fov)`
- `void moveCamera(const glm::vec3& v)`
- `void orbitCamera(float dTheta, float dPhi)`

Note that the `camera.move(v)` function call which is defined in Camera class is doing the actual math and calculation, over here the result is being sent into GPU via shader uniforms `camera.params`. 

Example: 

```c
void moveCamera(const glm::vec3& v) {
  camera.move(v);
  glBindBuffer(GL_UNIFORM_BUFFER, cameraUBO);
  glBufferSubData(GL_UNIFORM_BUFFER, 0, sizeof(CameraBlock), &camera.params);
  glBindBuffer(GL_UNIFORM_BUFFER, 0);
  clear_flag = true;
}
```

Few getter/setter functions:
```c
  RenderMode getRenderMode() const { return mode; }

  void setRenderMode(const RenderMode& mode) {
    this->mode = mode;
    clear();
  }

  Integrator getIntegrator() const { return integrator; }

  void setIntegrator(const Integrator& integrator) {
    this->integrator = integrator;
    clear();
  }
```

### Send Scene Data to Shader
Note that `setSceneType()` will be the one that sends scene data into OpenGL.

```c
  SceneType getSceneType() const { return scene_type; }

  void setSceneType(const SceneType& scene_type) {
    this->scene_type = scene_type;

    // recreate scene
    scene.setScene(scene_type);

    // send scene data
    glBindBuffer(GL_UNIFORM_BUFFER, sceneUBO);
    glBufferSubData(GL_UNIFORM_BUFFER, 0, sizeof(SceneBlock), &scene.block);
    glBindBuffer(GL_UNIFORM_BUFFER, 0);

    clear();
  }
```

## Renderer::render()
`render()` function calls `glViewport()` first, then it switches by **RenderMode** enum - here we focus on mode **Render**. 

Then we bind FBO by `glBindFramebuffer(GL_FRAMEBUFFER, accumFBO)`, which will get the drawing ready. 

Then it switches by **Integrator** enum - here we focus on **PT**: `rectangle.draw(pt_shader)`.

Note that the draw function is from `Rectangle` class, which will activate the passed-in shader object and basically draw the quad onto the screen with its fragment shader that doing the real path tracing.

```c
// from rectangle.h
  void draw(const Shader& shader) const {
    shader.activate(); // glUseProgram(program)

    glBindVertexArray(VAO); // bind VAO of the quad
    glDrawElements(GL_TRIANGLES, 6, GL_UNSIGNED_INT, 0);
    glBindVertexArray(0); // unbind

    shader.deactivate();
  }
```

This will draw one pass of the path tracing result, which equals to **sample once of the rendering equation**. 

Then it increments the samples count by one each loop: `samples++`

Remember to unbind FBO by `glBindFramebuffer(GL_FRAMEBUFFER, 0)`

Because we are doing **progressive rendering** here, the accumFBO - just as its name - is accumulating all sampled path tracing frames and in the end it will divide the sum by sample count and draw to the output shader:

```c
// output
output_shader.setUniform("samplesInv", 1.0f / samples);
rectangle.draw(output_shader);
```

## Render::clear()
```c
  void clear() {
    // clear accumTexture
    glBindTexture(GL_TEXTURE_2D, accumTexture);
    std::vector<GLfloat> data(3 * global.resolution.x * global.resolution.y);
    glTexSubImage2D(GL_TEXTURE_2D, 0, 0, 0, global.resolution.x,
                    global.resolution.y, GL_RGB, GL_FLOAT, data.data());
    glBindTexture(GL_TEXTURE_2D, 0);

    // update texture uniforms
    pt_shader.setUniformTexture("accumTexture", accumTexture, 0);
    pt_nee_shader.setUniformTexture("accumTexture", accumTexture, 0);
    bdpt_shader.setUniformTexture("accumTexture", accumTexture, 0);
    output_shader.setUniformTexture("accumTexture", accumTexture, 0);

    // reset samples
    samples = 0;
  }
```

# shader.h
It defines the infrastructure class `Shader` to configure OpenGL shader objects.

## Shader()
Constructor is taking in **shader file paths**, then it will **compile** and **link** the shaders. These are all standard boilerplate code.

### Compile shaders

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

### Link shaders

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

Helper functions to destroy, active/deactive shaders:
```c
void destroy() {
  glDeleteShader(vertex_shader);
  glDeleteShader(fragment_shader);
  glDeleteProgram(program);
}
void activate() const { glUseProgram(program); }
void deactivate() const { glUseProgram(0); }
```

### Set Uniforms Functions
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
Define the `Rectangle` class to prepare and draw the screen quad. This is basically the only OpenGL geometry we need to construct and send to GPU.

## Rectangle()
Define vertices to draw the screen quad and send all the related objects to OpenGL.
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
```

# scene.h
`Scene` class contains definition of the Cornell Box. It is a pure C++ class, data are packed and later sent into OpenGL by `Renderer`.

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

It defines several **memory-aligned struct** to **pack all the scene description data**:
-  `Primitive`
-  `Material`
-  `Light`
  

`global.frag` shader has the corresponding definitions of those structs:
```c
struct Material {
    int brdf_type;
    vec3 kd;
    vec3 le;
};

struct Primitive {
    int id;
    int type;
    vec3 center;
    float radius;
    vec3 leftCornerPoint;
    vec3 up;
    vec3 right;
    int material_id;
};

struct Light {
    int primID;
    vec3 le;
};
```

`SceneBlock` will be the top level container contains all data packs above.

`uniform.frag` shader has the corresponding definitions of `SceneBlock`'s **uniform block layout**, to utilize UBO - Uniform buffer objects:

- More to learn at [Advanced GLSL - Uniform buffer objects](https://learnopengl.com/Advanced-OpenGL/Advanced-GLSL)

```c
const int MAX_N_MATERIALS = 100;
const int MAX_N_PRIMITIVES = 100;
const int MAX_N_LIGHTS = 100;

layout(std140) uniform SceneBlock {
  int n_materials;
  int n_primitives;
  int n_lights;
  Material materials[MAX_N_MATERIALS];
  Primitive primitives[MAX_N_PRIMITIVES];
  Light lights[MAX_N_LIGHTS];
};
```



`setScene()` will clear the `SceneBlock`, and based on switch cases of scene type recreates the scene data. Here we focus on `SceneType.Original`, which calls `setupCornellBoxOriginal()`. 

Afterward it will `init()` the scene. `setupCornellBoxOriginal()` will call a lot of `addPrimitive()` and `addMaterial()` and keep track of the `n_materials` `n_primitives` and `n_lights`. These integers are used as id and assigned for all the corresponding scene elements - prims(geo) or lights.

`setupCornellBoxOriginal()` will create and fill `SceneBlock` with the required Primitive, Material, Light for the scene.

```c
class Scene {
 private:
  void setupCornellBoxOriginal() //...
  // ...

  void init() {
    // set primitive id
    for (int i = 0; i < n_primitives; ++i) {
      block.primitives[i].id = i;
    }

    // set lights
    int n_lights = 0;
    for (int i = 0; i < n_primitives; ++i) {
      const Primitive& primitive = block.primitives[i];
      const Material& material = block.materials[primitive.material_id];
      if (material.le != glm::vec3(0)) {
        Light light;
        light.primID = primitive.id;
        light.le = material.le;

        block.lights[n_lights] = light;
        n_lights++;
      }
    }

    // set number of materials, primitives, lights
    block.n_materials = n_materials;
    block.n_primitives = n_primitives;
    block.n_lights = n_lights;
  }

  void clear() {
    n_primitives = 0;
    n_materials = 0;
  }

 public:
  int n_primitives;
  int n_materials;
  SceneBlock block;

  void addPrimitive(const Primitive& primitive) {
    block.primitives[n_primitives] = primitive;
    n_primitives++;
  }

  void addMaterial(const Material& material) {
    block.materials[n_materials] = material;
    n_materials++;
  }

  static Primitive createSphere(const glm::vec3& center, float radius) // ...

  static Primitive createPlane(const glm::vec3& leftCornerPoint, const glm::vec3& right, const glm::vec3& up) // ...

  static Material createDiffuse(const glm::vec3& kd) // ...

  static Material createMirror(const glm::vec3& kd) // ...

  static Material createGlass(const glm::vec3& kd) // ...

  static Material createLight(const glm::vec3& le) // ...

  Scene() : n_primitives(0), n_materials(0) // ...

  void setScene(const SceneType& scene_type) // ...
};
```

# camera.h
For defining `Camera` class. Also a pure C++ class, data are packed and later sent into OpenGL by `Renderer`.

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
```

```c
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

  void setFOV(float fov) //...

  void move(const glm::vec3& v) //...

  void orbit(float dTheta, float dPhi) //...
};
```

`uniform.frag` shader has the corresponding definitions of `CameraBlock`'s **uniform block layout**

```c
layout(std140) uniform CameraBlock {
  vec3 camPos;
  vec3 camForward;
  vec3 camRight;
  vec3 camUp;
  float a;
} camera;
```

---

Moving onto next note to break down the pace tracing implementation on the GLSL shader side. :)

