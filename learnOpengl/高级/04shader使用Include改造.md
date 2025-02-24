现在理解一下我们要做什么
我们的`shader`代码过于冗长，我希望可以创建一个`commonLight.glsl`，专门用来放光照的结构声明以及计算
然后我们去`phong.frag`使用`include`
就像我们正常的`.cpp`用`include`得到`.h`一样

![输入图片说明](/imgs/2025-02-24/qFPC4IMXHcmwgY0w.png)

观察一下现在的文件夹结构，我们希望`advanced`下的`phong.frag`可以`include`到`common`文件夹下的`commonLight.glsl`

在`phong.frag`这样写就好了`#include "../common/commonLight.glsl"`
dan
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTEwMDA5NjY0MTMsLTIwODg3NDY2MTJdfQ
==
-->