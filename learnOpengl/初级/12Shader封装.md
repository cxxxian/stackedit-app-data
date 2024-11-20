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
完成shader构造函数，大迁移！：
```
Shader::Shader(const char* vertexPath, const char* fragmentPath)
{
	//声明装入shader代码字符串的两个string
	std::string vertexCode;
	std::string fragmentCode;
	//声明用于读取vs和fs文件的inFileStream
	std::ifstream vShaderFile;
	std::ifstream fShaderFile;

	//保证ifstream遇到问题时可以抛出异常
	vShaderFile.exceptions(std::ifstream::failbit | std::ifstream::badbit);
	fShaderFile.exceptions(std::ifstream::failbit | std::ifstream::badbit);
	try {
		//1 打开文件
		vShaderFile.open(vertexPath);
		fShaderFile.open(fragmentPath);

		//2 将文件输入流当中的字符串输入到stringStream里面
		std::stringstream vShaderStream, fShaderStream;
		vShaderStream << vShaderFile.rdbuf();
		fShaderStream << fShaderFile.rdbuf();

		//3 关闭文件
		vShaderFile.close();
		fShaderFile.close();

		//4 将字符串从stringStream当中读取出来，转化到code String当中
		vertexCode = vShaderStream.str();
		fragmentCode = fShaderStream.str();
	}catch(std::ifstream::failure& e){
		std::cout << "ERROR: Shader File Error: " << e.what() << std::endl;
	}

    const char* vertexShaderSource = vertexCode.c_str();
    const char* fragmentShaderSource = fragmentCode.c_str();

    //1 创建shader程序
    GLuint vertex, fragment;
    vertex = glCreateShader(GL_VERTEX_SHADER);
    fragment = glCreateShader(GL_FRAGMENT_SHADER);

    //2 为shader程序输入shader代码
    //此时无需告诉字符串长度，之间填NULL
    //因为我们用\0结尾，知道结尾在哪
    glShaderSource(vertex, 1, &vertexShaderSource, NULL);
    glShaderSource(fragment, 1, &fragmentShaderSource, NULL);

    int success = 0;
    char infoLog[1024];
    //3 执行shader代码编译
    glCompileShader(vertex);
    //检查vertex编译结果
    glGetShaderiv(vertex, GL_COMPILE_STATUS, &success);
    if (!success) {
        glGetShaderInfoLog(vertex, 1024, NULL, infoLog);
        std::cout << "Error: SHADER COMPILE ERROR -- Vertex" << "\n" << infoLog << std::endl;
    }
    glCompileShader(fragment);
    //检查fragment编译结果
    glGetShaderiv(fragment, GL_COMPILE_STATUS, &success);
    if (!success) {
        glGetShaderInfoLog(fragment, 1024, NULL, infoLog);
        std::cout << "Error: SHADER COMPILE ERROR -- Fragment" << "\n" << infoLog << std::endl;
    }

    //4 创建一个program壳子
    mProgram = glCreateProgram();

    //5 将vs和fs编译好的结果放到program这个壳子里面
    glAttachShader(mProgram, vertex);
    glAttachShader(mProgram, fragment);

    //6 执行program的连接操作，形成最终可执行shader程序
    glLinkProgram(mProgram);
    //检查链接错误
    glGetProgramiv(mProgram, GL_LINK_STATUS, &success);
    if (!success) {
        glGetProgramInfoLog(mProgram, 1024, NULL, infoLog);
        std::cout << "Error: SHADER LINK ERROR" << "\n" << infoLog << std::endl;
    }

    //清理
    glDeleteShader(vertex);
    glDeleteShader(fragment);
}
```
完成以上封装之后，我们直接在main.cpp中
`Shader *shader = nullptr;`
然后调用shader构造函数
```
void prepareShader() {
    shader = new Shader("assets/shaders/vertex.glsl", "assets/shaders/fragment.glsl");
}
```
使用刚刚写的begin和end，使用end这样我们就可以使用多个shader
```
void render(){
    //执行opengl画布清理操作
    glClear(GL_COLOR_BUFFER_BIT);

    //1 绑定当前的program
    shader->begin();
    //2 绑定当前的vao
    glBindVertexArray(vao);
    //3 发出绘制指令
    //glDrawArrays(GL_LINE_STRIP, 0, 6);
    glDrawElements(GL_TRIANGLES, 6, GL_UNSIGNED_INT, 0);

    shader->end();
}
```
以下为错误检查的封装
```
void Shader::checkShaderErrors(GLuint target, std::string type)
{
    int success = 0;
    char infoLog[1024];
    if (type == "COMPILE") {
        glGetShaderiv(target, GL_COMPILE_STATUS, &success);
        if (!success) {
            glGetShaderInfoLog(target, 1024, NULL, infoLog);
            std::cout << "Error: SHADER COMPILE ERROR" << "\n" << infoLog << std::endl;
        }
        glCompileShader(target);
        //检查fragment编译结果
        glGetShaderiv(target, GL_COMPILE_STATUS, &success);
    }
    else if (type == "LINK") {
        glGetProgramiv(target, GL_LINK_STATUS, &success);
        if (!success) {
            glGetProgramInfoLog(target, 1024, NULL, infoLog);
            std::cout << "Error: SHADER LINK ERROR" << "\n" << infoLog << std::endl;
        }
    }
    else {
        std::cout << "Error: Check shader errors Type is wrong" << std::endl;
    }
}
```
<!--stackedit_data:
eyJoaXN0b3J5IjpbNTM5Njc0NTE0XX0=
-->