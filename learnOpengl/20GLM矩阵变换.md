目标是变换三角形，因为三个顶点会旋转同样的角度，所以可以共用一个旋转矩阵，即声明`uniform mat4 transform;`
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
    vec4 postion = vec4(aPos, 1.0);
    position = transform * position;
    gl_Position = position;
    color = aColor;
    uv = aUV;
}
```
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTczODA3ODEyXX0=
-->