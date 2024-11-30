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
    ImGui_ImplOpenGL3_Init("version 460");
}
```
回到`application.h`中声明函数
```cpp
GLFWwindow* getWindow() const { return mWindow; }
```

## 渲染
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE2NjMwNDU4NCwtNDkxMTM2NDQzLDIxMT
A4MDA5MTBdfQ==
-->