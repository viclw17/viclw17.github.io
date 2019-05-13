---
title: OpenGL Project Setup
date: 2019-05-10
tags:
- OpenGL
---
<!-- <img src="{{ site.url }}/images/raytracing-texture.jpg" width="640"  style="display:block; margin:auto;">
<br> -->
I had quite a journey setting up the environment for OpenGL development. Lots of research was done while I was solving all the problems and here I documented my version of approach.

# Resources

I've been mainly using Visual Studios to work on my previous Raytracing project since it was raw C++ implementation. As OpenGL is a graphic API that has been integrated into MacOS, I started with setting the project in Xcode.

The tutorial I've been following is https://github.com/tomdalling/opengl-series which covers the basics in details with code base supporting multiple platforms(MacOS, Windows, Linux). Here I base my practice on its first chapter - **01_project_skeleton**.

The website http://learnopengl.com is however by far the most well-rounded one for me, and I will constantly referring to it.

# Xcode
## Code Breakdown
### Includes
```c
#include "platform.hpp"

// third-party libraries
#include <GL/glew.h>
#include <GLFW/glfw3.h>
#include <glm/glm.hpp>

// standard c++ libraries
#include <cassert>
#include <iostream>
#include <stdexcept>
#include <cmath>

#include "Program.h"
```
The code is using third-party libraries including

- [GLEW](http://glew.sourceforge.net/) (Graphics Library Extension Wrangler),
- [GLFW](https://www.glfw.org/) (Graphics Library FrameWork), and
- [GLM](https://glm.g-truc.net/0.9.9/index.html) (OpenGL Mathematics).

Note that in the tutorial code of http://learnopengl.com , instead of GLEW it is now using [GLAD](https://glad.dav1d.de/) as an alternative.

Among them, GLFW has to be built for the corresponding OS. This can be done with [CMake](https://cmake.org/) and details can be found [here](https://learnopengl.com/Getting-started/Creating-a-window). A ```libglfw3.a``` file will be generated if on MacOS (and ```glfw3.lib``` if on Windows).

To include all of them into project, best practice is to group GLEW/GLFW/GLM header files into a folder **Includes**, and put ```libglfw3.a``` into a folder **Libs**.

```platform.hpp``` contains declaration of ```std::string ResourcePath(std::string fileName);```, an extra function used to get the path of shader files on MacOS. The corresponding ```platform_osx.mm``` contains its defination in objective-c.

The code puts functionalities of shader loading/compiling into the **Shader.h/Shader.cpp** and shader program linking into **Program.h/Program.cpp**. Hence we include ```Program.h``` here.

To get the includes working properly in Xcode project, we have to set the path in Project Setting - Search Paths:

<img src="{{ site.url }}/images/OpenGL/search-paths.jpg" width="640"  style="display:block; margin:auto;">
<br>

### main.cpp
Here I removed most of error-checking code to get a minimal look of the whole setup code, just for future reference. However they are very important in practice to help locating problems.

```c
// constants
const glm::vec2 SCREEN_SIZE(400,400);

// globals
GLFWwindow* gWindow = NULL;
GLuint gVAO = 0;
GLuint gVBO = 0;
tdogl::Program* gProgram = NULL;

// loads the vertex shader and fragment shader, and
// links them to make the global gProgram
static void LoadShaders(){
    std::vector<tdogl::Shader> shaders;
    shaders.push_back(tdogl::Shader::shaderFromFile(
    ResourcePath("vertex-shader.txt"), GL_VERTEX_SHADER));
    shaders.push_back(tdogl::Shader::shaderFromFile(
    ResourcePath("fragment-shader.txt"), GL_FRAGMENT_SHADER));
	gProgram = new tdogl::Program(shaders);
}

static void LoadTriangle(){
    glGenVertexArrays(1, &gVAO);
    glBindVertexArray(gVAO);
    glGenBuffers(1, &gVBO);
    glBindBuffer(GL_ARRAY_BUFFER, gVBO);

    GLfloat vertexData[] = {
        // X       Y     Z
         0.000f,  0.5f, 0.0f,
        -0.577f, -0.5f, 0.0f,
         0.577f, -0.5f, 0.0f,
    };

    glBufferData(GL_ARRAY_BUFFER, sizeof(vertexData), vertexData, GL_STATIC_DRAW);
    glEnableVertexAttribArray(gProgram->attrib("vert"));
    glVertexAttribPointer(gProgram->attrib("vert"), 3, GL_FLOAT, GL_FALSE, 0, NULL);

    glBindBuffer(GL_ARRAY_BUFFER,0);
    glBindVertexArray(0);
}
```

Render function to draw stuff.

```c
static void Render(){
    glClearColor(0,0,0,1);
    glClear(GL_COLOR_BUFFER_BIT);

    glUseProgram(gProgram->object());
    glBindVertexArray(gVAO);
	glDrawArrays(GL_TRIANGLES,0,3);

    glBindVertexArray(0);
	glUseProgram(0);

    glfwSwapBuffers(gWindow);
}
```

Main function to
- initialize GLFW and GLEW
- create window
- load shader program
- load geometry
- loop the ```Render()``` funtion.

```c
void AppMain(){
	// initialise GLFW
	glfwInit();

	glfwWindowHint(GLFW_OPENGL_FORWARD_COMPAT, GL_TRUE);
	glfwWindowHint(GLFW_OPENGL_PROFILE, GLFW_OPENGL_CORE_PROFILE);
	glfwWindowHint(GLFW_CONTEXT_VERSION_MAJOR,3);
	glfwWindowHint(GLFW_CONTEXT_VERSION_MINOR,2);

	gWindow = glfwCreateWindow((int)SCREEN_SIZE.x,(int)SCREEN_SIZE.y,"test",NULL,NULL);

	// GLFW settings
	// Before you can use the OpenGL API, you must have a current OpenGL context.
	glfwMakeContextCurrent(gWindow);

	// initialise GLEW
	glewExperimental = GL_TRUE; // stops glew crashing on OSX :-/
	glewInit();

	LoadShaders();
	LoadTriangle();

	while(!glfwWindowShouldClose(gWindow)){
	    glfwPollEvents();
	    Render();
	}
	glfwTerminate();
}

int main(int argc, char *argv[]){
    AppMain();
}
```

### Shader Class
Header:

```c
#include <GL/glew.h>
#include <string>

namespace tdogl {
	class Shader{
	public:
		static Shader shaderFromFile(const std::string& filePath, GLenum shaderType);
		Shader(const std::string& shaderCode, GLenum shaderType);
		GLuint object() const;
	private:
		GLuint _object;
		unsigned* _refCount;
		void _release();
	};
}
```
Source:

```c
#include "Shader.h"
#include <stdexcept>
#include <fstream>
#include <string>
#include <cassert>
#include <sstream>

using namespace tdogl;

Shader::Shader(const std::string& shaderCode, GLenum shaderType) :
_object(0),
_refCount(NULL)
{
    //create the shader object
    _object = glCreateShader(shaderType); // had bug here before!
    if(_object == 0)
        throw std::runtime_error("glCreateShader failed");

    //set the source code
    const char* code = shaderCode.c_str();
    glShaderSource(_object, 1, (const GLchar**)&code, NULL);

    //compile
    glCompileShader(_object);

    //throw exception if compile error occurred
    // ...

    _refCount = new unsigned;
    *_refCount = 1;
}

GLuint Shader::object() const {
    return _object;
}

Shader Shader::shaderFromFile(const std::string& filePath, GLenum shaderType) {
    //open file
    std::ifstream f;
    f.open(filePath.c_str(), std::ios::in | std::ios::binary);

    if(!f.is_open()){
        throw std::runtime_error(std::string("Failed to open file: ") + filePath);
    }

    //read whole file into stringstream buffer
    std::stringstream buffer;
    buffer << f.rdbuf();

    //return new shader
    Shader shader(buffer.str(), shaderType);
    return shader;
}

void Shader::_release() {
    assert(_refCount && *_refCount > 0);
    *_refCount -= 1;
    if(*_refCount == 0){
        glDeleteShader(_object); _object = 0;
        delete _refCount; _refCount = NULL;
    }
}

```

### Program Class
Header:

```c
#include "Shader.h"
#include <vector>

namespace tdogl {
	class Program{
	public:
		Program(const std::vector<Shader>& shaders);
		GLuint object() const;
		GLint attrib(const GLchar* attribName) const;
	private:
		GLuint _object;
		Program(const Program&);
	};
}
```

Source:

```c
#include "Program.h"
#include <stdexcept>

using namespace tdogl;

Program::Program(const std::vector<Shader>& shaders) :
_object(0)
{
    if(shaders.size() <= 0)
        throw std::runtime_error("No shaders were provided to create the program");

    //create the program object
    _object = glCreateProgram();
    if(_object == 0)
        throw std::runtime_error("glCreateProgram failed");

    //attach all the shaders
    for(unsigned i = 0; i < shaders.size(); ++i)
        glAttachShader(_object, shaders[i].object());

    //link the shaders together
    glLinkProgram(_object);

    //detach all the shaders
    for(unsigned i = 0; i < shaders.size(); ++i)
        glDetachShader(_object, shaders[i].object());

    //throw exception if linking failed
    // ...
}

GLuint Program::object() const {
    return _object;
}

GLint Program::attrib(const GLchar* attribName) const {
    if(!attribName)
        throw std::runtime_error("attribName was NULL");

    GLint attrib = glGetAttribLocation(_object, attribName);
    if(attrib == -1)
        throw std::runtime_error(std::string("Program attribute not found: ") + attribName);

    return attrib;
}
```

### Shader Files
#### Vertex Shader
```c
#version 150
in vec3 vert;
void main() {
    // does not alter the verticies at all
    gl_Position = vec4(vert, 1);
}
```
#### Fragment Shader
```c
#version 150
out vec4 finalColor;
void main() {
    //set every drawn pixel to red
    finalColor = vec4(1.0, 0, 0, 1.0);
}
```

## Debug
### Apple Mach-O Linker (ld) Error Group
This is caused by failure on linking libraries or source files. Need to check Build Phase of build target.

If the errors are about **glfw functions**, it's because we didn't link with ```libglfw3.a``` properly. If they are about **other OpenGL functions** that is because we have to explicitly link with certain frameworks there:

<img src="{{ site.url }}/images/OpenGL/libs.jpg" width="640"  style="display:block; margin:auto;">
<br>

Also make sure all the source files (all .cpp including the .mm) are added into Compile Sources of the build target.

If there are errors about **glew functions**, that is because we also need to explicitly add ```glew.c``` file to compile,

<img src="{{ site.url }}/images/OpenGL/src.jpg" width="640"  style="display:block; margin:auto;">
<br>

Another option is to just include it in ```main.cpp```

```c
#include <GL/glew.h>
#include <glew.c> // add this
#include <GLFW/glfw3.h>
#include <glm/glm.hpp>
```

### Thread 1: signal SIGABRT
At this point I can pass the compiling but still failed on running the build.

The program broke at function ```Shader::shaderFromFile()```, and I found the path where the program reading my shader files is actually to the wrong folder(thanks to breakpoint). So I set the new ```DerivedData``` folder and put shader files right by the executable inside ```DerivedDate/.../Build/Products```.

<img src="{{ site.url }}/images/OpenGL/folder.jpg" width="320"  style="display:block; margin:auto;">
<br>

This way the function successfully located the shader files and the program proceeded.

### Thread 1: EXC_BAD_ACCESS (code=1, address=0x0)
I also got this error at some point. I made sure the shader files are successfully located and shader program is successfully created. I then found the error actually happened when I was calling ```glCreateShader()``` in ```Shader.cpp```.

<img src="{{ site.url }}/images/OpenGL/error.jpg" width="320"  style="display:block; margin:auto;">
<br>

It turns out I missed the line ```glfwMakeContextCurrent(gWindow);``` in ```main.cpp```, so I was [calling GL functions before I have a current GL context!](https://stackoverflow.com/questions/55906938/code-error-thread-1-exc-bad-access-code-1-address-0x0)

## Result
Finally it compiled and ran successfully!

<img src="{{ site.url }}/images/OpenGL/test.png" width="480"  style="display:block; margin:auto;">
<br>

# Visual Studios
TBC
