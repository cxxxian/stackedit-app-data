# 配置

![输入图片说明](/imgs/2024-11-30/n6Og1RfmmEtEIifr.png)

引入`imgui`需要的文件，以及建立`CmakeLists`
`CmakeLists`如下：编译出`imguilib`
```
file(GLOB_RECURSE IMGUI ./  *.cpp)
add_library(imguilib ${IMGUI} )
```

然后到外面总的`CmakeLists`中添加：
```
add_subdirectory(wrapper)
add_subdirectory(application)
add_subdirectory(glframework)
add_subdirectory(imgui)

#本主程序文件及输出程序名称
add_executable(openglStudy "main.cpp" "glad.c")

target_link_libraries(openglStudy glfw3.lib wrapper app fw imguilib)
```
# 使用
## 初始化
初始化`IMGUI`，但由于先前我们并没有设计获取当前窗口的函数
```cpp
void initIMGUI() {
    ImGui::CreateContext();//创建imgui上下文
    ImGui::StyleColorsDark();

    //设置ImGui与GLFW和OpenGL的绑定
    ImGui_ImplGlfw_InitForOpenGL(app->getWindow(), true);
    ImGui_ImplOpenGL3_Init("#version 460");
}
```
回到`application.h`中声明函数
```cpp
GLFWwindow* getWindow() const { return mWindow; }
```

## 渲染
在`main.cpp`中设计一个用来渲染`IMGUI`的方法
```cpp
void renderIMGUI() {
    //1 开启当前的IMGUI渲染
    ImGui_ImplOpenGL3_NewFrame();
    ImGui_ImplGlfw_NewFrame();
    ImGui::NewFrame();

    //2 决定当前的GUI上面有哪些控件，从上到下
    ImGui::Begin("Hello, world");
    ImGui::End();

    //3 执行UI渲染
    ImGui::Render();
    //获取当前窗体的宽高
    int display_w, display_y;
    glfwGetFramebufferSize(app->getWindow(), &display_w, &display_y);
    //重置视口大小
    glViewport(0, 0, display_w, display_y);

    ImGui_ImplOpenGL3_RenderDrawData(ImGui::GetDrawData());
}
```
并将其用在main函数的循环中：
```cpp
while (app->update()) {

    cameraControl->update();
    renderer->render(meshes, camera, dirLight, pointLights, spotLight, ambLight);
    renderIMGUI();
}
```

![输入图片说明](/imgs/2024-11-30/wLgTEY4pIvNf0c73.png)

```cpp
glm::vec3 clearColor{};

void renderIMGUI() {
    ...
    //2 决定当前的GUI上面有哪些控件，从上到下
    ImGui::Begin("Hello, world");
    ImGui::Text("ChangeColor Demo");
    ImGui::Button("Test Button", ImVec2(40, 20));
    ImGui::ColorEdit3("Clear Color", (float*)&clearColor);
    ImGui::End();
	...
}
```
多添加一些功能，在`main.cpp`的全局设计一个clearColor，并传入`ColorEdit3`作为参数。
我们希望这个功能可以通过`IMGUI`更改背景色
在`renderer.h`中声明并在`cpp`中实现
```cpp
void Renderer::setClearColor(glm::vec3 color)
{
	glClearColor(color.r, color.g, color.b, 1.0);
}
```
最后我们回到`main.cpp`的`main`函数中的循环中调用即可。
此处是分开的，通过`IMGUI`去调节颜色给`clearColor`赋值，然后再把`clearColor`传到`renderer`的`setClearColor`方法中进行背景色的设置
```cpp
while (app->update()) {

    cameraControl->update();
    renderer->setClearColor(clearColor);
    renderer->render(meshes, camera, dirLight, pointLights, spotLight, ambLight);
    renderIMGUI();
}
```

![输入图片说明](/imgs/2024-11-30/70CkLjhicG3iiS8v.png)
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTkzMTI2NDE1MiwtMTIwMzg1MzA0MCwtMT
Y2MzA0NTg0LC00OTExMzY0NDMsMjExMDgwMDkxMF19
-->