# GLSL
![输入图片说明](/imgs/2024-10-23/qlG2osoa38B3S6Td.png)
![输入图片说明](/imgs/2024-10-23/vXuEc12KrJwJBGhd.png)
![输入图片说明](/imgs/2024-10-23/V7oV4iQX0whKxbHO.png)
# shader变量
![输入图片说明](/imgs/2024-10-23/kr29NmybGr3lCW5x.png)
## 输入
![输入图片说明](/imgs/2024-10-23/WCQvMQcVIne89dCH.png)
此方法可以动态获取属性编号，但是这样的话需要将mProgram公开（不推荐）
![输入图片说明](/imgs/2024-10-23/1DC6tig6fG5BKJyH.png)
属性编号是从上到下分配的
两种顺序会得到不同的location值
posLocation = 0
colorLocation = 1
```
in vec3 aPos;
in vec3 aColor;
```
posLocation = 1
colorLocation = 0
```
in vec3 aColor;
in vec3 aPos;
```
## uniform
![输入图片说明](/imgs/2024-10-23/opnmygIOSyWXHtDl.png)
![输入图片说明](/imgs/2024-10-23/03DSXGjREaSWkO1J.png)
![输入图片说明](/imgs/2024-10-23/im4pWGswALBlNcNu.png)
![输入图片说明](/imgs/2024-10-23/uZS1RM7F7Dry3u65.png)
eg.
```
uniform int a;
glUniform1i(。。。);
```

### uniform的使用
在shader.h中声明函数用来设置参数（制作此方法的主要原因是因为我们不想把mProgram暴露为public，所以制作此函数用来设置value给mProgram）
```
void setFloat(std::string& name, float value);
```
在cpp实现如下
```
void Shader::setFloat(std::string& name, float value)
{
    //1 通过名称拿到Uniform变量位置location
    GLuint location = GL_CALL(glGetUniformLocation(mProgram, name.c_str()));

    //2 通过location更新Uniform变量的值
    GL_CALL(glUniform1f(location, value));
}
```
为了实现三角形忽明忽暗的效果，我们可以在vs或者fs里操作，此处
```
#version 460 core
layout (location = 0) in vec3 aPos;
layout (location = 1) in vec3 aColor;
out vec3 color;

uniform float time;

void main()
{
    gl_Position = vec4(aPos.x, aPos.y, aPos.z, 1.0);
    color = aColor * (sin(time) + 1.0) / 2.0;
}
```

<!--stackedit_data:
eyJoaXN0b3J5IjpbMzUwMzEyNDMzLC05MTgxMzA5MDIsLTEzND
A3Njg5NDUsLTMyODM4MTQ2NSw0NjI2MzIwOTQsMjA5NTA2NjA0
NywxNzk3ODU1MDUyXX0=
-->