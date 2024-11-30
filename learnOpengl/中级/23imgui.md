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

GLFWwindow* getWindow() const { return mWindow; }
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTcwMDU2Njk4NF19
-->