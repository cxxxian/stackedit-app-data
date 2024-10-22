glsl插件
![输入图片说明](/imgs/2024-10-22/bTMdSSuMIuegkk1k.png)
新建shader.h:
```
#pragma once
#include "core.h"

class Shader {
public:
	Shader();
	~Shader();

	void begin();//开始使用当前shader
	void end();//结束使用当前shader
private:
	GLuint mProgram{ 0 };
};
```
shader.cpp:
```
#include "shader.h"
#include "../wrapper/checkError.h"

Shader::Shader()
{}
Shader::~Shader()
{}
void Shader::begin()
{
	GL_CALL(glUseProgram(mProgram));
}

void Shader::end()
{
	GL_CALL(glUseProgram(0));
}

```
分别建立vertex.glsl和fragment.glsl
```
#version 460 core
layout (location = 0) in vec3 aPos;
layout (location = 1) in vec3 aColor;
out vec3 color;
void main()
{
    gl_Position = vec4(aPos.x, aPos.y, aPos.z, 1.0);
    color = aColor;
    }
```
```
#version 330 core
out vec4 FragColor;
in vec3 color;
void main()
{
    FragColor = vec4(color, 1.0f);
}
```

<!--stackedit_data:
eyJoaXN0b3J5IjpbMTgyNDIwOTk4LDE4OTMwNDM2NzIsLTMwMz
U0NTA4N119
-->