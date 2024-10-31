目标是变换三角形，在`vertex.glsl`中操作顶点。
因为三个顶点会旋转同样的角度，所以可以共用一个旋转矩阵，即声明`uniform mat4 transform;`，`position = transform * position`即可以做到旋转变化
```
#version 460 core
layout (location = 0) in vec3 aPos;
layout (location = 1) in vec3 aColor;
layout (location = 2) in vec2 aUV;
out vec3 color;
out vec2 uv;

uniform mat4 transform;

void main()
{
    vec4 position = vec4(aPos, 1.0);
    position = transform * position;
    gl_Position = position;
    color = aColor;
    uv = aUV;
}
```
在shader.cpp中制作一个`setMatrix()`用来传输矩阵到shader中
```
void Shader::setMatrix4x4(const std::string& name, glm::mat4 value)
{
    //1 通过名称拿到Uniform变量位置location
    GLuint location = GL_CALL(glGetUniformLocation(mProgram, name.c_str()));
    //2 通过location更新Uniform变量的值
    //第二个参数：传n个矩阵；第三个参数：传的是否需要转置；第四个参数：指向value的指针
    glUniformMatrix4fv(location, 1, GL_FALSE, glm::value_ptr(value));
}
```
OpenGL的矩阵存储为列优先，GLM也恰好是列优先，所以不需要转置

回到main.cpp中，先声明一个单位矩阵transform
```
glm::mat4 transform(1.0);
```
制作一个函数用来做旋转操作
```
void doTransform() {
    //构建一个旋转矩阵，绕z轴旋转45度
    //rotate必须得到一个float类型的参数，跟template有关系
    transform = glm::rotate(glm::mat4(1.0f), glm::radians(90.0f), glm::vec3(0.0, 0.0, 1.0));
}
```

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTQ1OTA3NDYyOSwtMTgyMzg4MjQzOSwxMz
YxNTQxMjA3LC0xODc2NjQ2NDg5LC0xNTQ5NzU5NTgyLC03Mzgw
NzgxMl19
-->