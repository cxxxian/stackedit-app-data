现在理解一下我们要做什么
我们的`shader`代码过于冗长，我希望可以创建一个`commonLight.glsl`，专门用来放光照的结构声明以及计算
然后我们去`phong.frag`使用`include`
就像我们正常的`.cpp`用`include`得到`.h`一样
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTEwOTE5NTA1MzcsLTIwODg3NDY2MTJdfQ
==
-->