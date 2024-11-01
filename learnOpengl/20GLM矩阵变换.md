# 矩阵变换
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
## 矩阵旋转
制作一个函数用来做旋转操作
```
void doRotationTransform() {
    //构建一个旋转矩阵，绕z轴旋转45度
    //rotate必须得到一个float类型的参数，跟template有关系
    //且rotate传的是弧度而不是角度，需要调用glm::radians()，而且radians()也是模板函数，需要传入float
    transform = glm::rotate(glm::mat4(1.0f), glm::radians(90.0f), glm::vec3(0.0, 0.0, 1.0));
}
```
在`main`中使用`doTransform()`进行旋转操作，只旋转一次所以把方法放在`while`外面
```
int main() {

    if (!app->init(800, 600)) {
        return -1;
    }
    app->setResizeCallback(OnResize);
    app->setKeyBoardCallback(OnKey);
    //注册窗口变化监听
    //glfwSetFramebufferSizeCallback(window, framebufferSizeCallback);
    //glfwSetKeyCallback(window, KeyCallBack);

    //设置opengl视口以及清理颜色
    glViewport(0, 0, 800, 600);
    glClearColor(0.2f, 0.3f, 0.3f, 1.0f);

    prepareShader();
    //prepareInterleavedBuffer();
    //prepareVAOForGLTriangles();
    prepareVAO();
    prepareTexture();

    doRotationTransform();

    while (app->update()) {
        render();
        //渲染操作
    }
    app->destroy();
    delete texture;
    return 0;
}
```
最后去`render`中通过`setMatrix4x4()`把`transform`传给`vertex.glsl`
这里把`shader->setMatrix4x4`放在render里面是因为正常在游戏中矩阵都是每一帧都更新的，需要每一帧都传入矩阵
```
void render(){
    //执行opengl画布清理操作
    glClear(GL_COLOR_BUFFER_BIT);

    //1 绑定当前的program
    shader->begin();

    shader->setInt("sampler", 0);//此处值为0是因为我们的纹理绑定在0号位上
    shader->setMatrix4x4("transform", transform);

    //2 绑定当前的vao
    glBindVertexArray(vao);
    //3 发出绘制指令
    //glDrawArrays(GL_LINE_STRIP, 0, 6);
    glDrawElements(GL_TRIANGLES, 6, GL_UNSIGNED_INT, 0);

    shader->end();
}
```
## 矩阵平移和缩放
制作平移和缩放操作。
效果分别为按照x轴平移0.5f，和在xy轴执行缩小0.5倍
```
void doTranslationTransform() {
    //构建一个旋转矩阵，绕z轴旋转45度
    //rotate必须得到一个float类型的参数，跟template有关系
    //且rotate传的是弧度而不是角度，需要调用glm::radians()，而且radians()也是模板函数，需要传入float
    transform = glm::translate(glm::mat4(1.0f), glm::vec3(0.5f, 0.0f, 0.0f));
}
void doScaleTransform() {
    //构建一个旋转矩阵，绕z轴旋转45度
    //rotate必须得到一个float类型的参数，跟template有关系
    //且rotate传的是弧度而不是角度，需要调用glm::radians()，而且radians()也是模板函数，需要传入float
    transform = glm::scale(glm::mat4(1.0f), glm::vec3(0.5f, 0.5f, 1.0f));
}
```
## 矩阵复合变换
矩阵复合变换
```
void doTransform() {

    glm::mat4 rotateMat = glm::rotate(glm::mat4(1.0f), glm::radians(45.0f), glm::vec3(0.0, 0.0, 1.0));
    glm::mat4 translateMat = glm::translate(glm::mat4(1.0f), glm::vec3(0.5f, 0.0f, 0.0f));
    // 先旋转后移动，从右到左看
    transform = translateMat * rotateMat;
}
```
## 持续旋转
制作一个`doRotation()`函数
```
float angle = 0.0f;
void doRotation() {
    angle += 2.0f;
    //每一帧都会“重新”构建一个旋转矩阵
    transform = glm::rotate(glm::mat4(1.0f), glm::radians(angle), glm::vec3(0.0, 0.0, 1.0));
}
```
在main函数中的while循环中调用它。
```
while (app->update()) {
        doRotation();
        render();
        //渲染操作
}
```
此处需要**注意的点**：虽然`angle`每次都累加`2.0f`，但它并不会转得越来越快，因为它每次都会从初始矩阵开始执行此旋转。
主要原因是：
在`vertex.glsl`中，我们每次都用`transform * position`，`transform`在此处的值是旋转矩阵，`position`为三角形一开始的位置，所以是每次都从初始矩阵开始执行此旋转。
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
# 叠加变换（重要）
![输入图片说明](/imgs/2024-11-01/WZ4LNnUOuxpqwReW.png)
第一个`rotate`方法的第一个参数为`glm::mat4(1.0f)`，说明是以单位矩阵为基础进行变换，其实也就相当于按照世界坐标系
第二个`translate`方法的第一个参数为`transform`，即是以`transform`的本地坐标进行变换
![输入图片说明](/imgs/2024-11-01/Wl5PQJFnJaYdMoQJ.png)
所以可以发现，此处一个**很重要的现象**：
需要区分于单纯的矩阵相乘，它平移后不会以原点进行旋转，而是按照自己本地坐标系的中心点为基准进行旋转
![输入图片说明](/imgs/2024-11-01/Okq7wJgSbUCt23Gp.png)
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTg0MDI4NTExNCwtNTYzODE1MDMzLDE5Mj
kwNjY4NzEsNjE5MzY4MzYwLDE2MDcyNTEwODIsMTUwNzQ1ODc3
MSwtNTU5Nzc4NTMxLDEzMTMxMDY4NjcsLTE4MjM4ODI0MzksMT
M2MTU0MTIwNywtMTg3NjY0NjQ4OSwtMTU0OTc1OTU4MiwtNzM4
MDc4MTJdfQ==
-->