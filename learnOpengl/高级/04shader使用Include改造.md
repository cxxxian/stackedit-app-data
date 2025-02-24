现在理解一下我们要做什么
我们的`shader`代码过于冗长，我希望可以创建一个`commonLight.glsl`，专门用来放光照的结构声明以及计算
然后我们去`phong.frag`使用`include`
就像我们正常的`.cpp`用`include`得到`.h`一样

![输入图片说明](/imgs/2025-02-24/qFPC4IMXHcmwgY0w.png)

观察一下现在的文件夹结构，我们希望advanced下的phong.frag可以include到`common`文件夹下的`commonLight.glsl`
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE1MzYwMjIwNzQsLTIwODg3NDY2MTJdfQ
==
-->