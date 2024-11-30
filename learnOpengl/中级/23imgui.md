![输入图片说明](/imgs/2024-11-30/n6Og1RfmmEtEIifr.png)

引入`imgui`需要的文件，以及建立`CmakeLists`
`CmakeLists`如下：编译出`imguilib`
```
file(GLOB_RECURSE IMGUI ./  *.cpp)
add_library(imguilib ${IMGUI} )
```
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTYxMjgyMTMyN119
-->