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
f
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE4ODk5NTc3MjksLTMwMzU0NTA4N119
-->