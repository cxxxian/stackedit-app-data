# Texture
对着这个一个一个来看
`Format`对应的是下面的`Compression Setting`的压缩格式
同时`Resource Size`是大小，会随着压缩的方式变化
当前选择的是`BC7`，精度会比较高，`Default`会低一点，里面也有一些`mask`，`Alpha`，`HDR`可以选择
`Compress Without Alpha`顾名思义就是不需要`Alpha`的时候可以勾选
以及`sRGB`，不勾选的时候就是线性颜色空间（`Linear
 Color`）这个同时需要去使用贴图的地方选对应的格式，如果在材质中使用贴图的时候选择了`Color`颜色空间，但是我们在`texture`把`sRGB`取消勾选了，此时就会报错，需要对应

![输入图片说明](/imgs/2026-08-11/nfeJ8fFm3jGlUmXg.png)
<!--stackedit_data:
eyJoaXN0b3J5IjpbMjAyOTQyMjU0MCwxNTc0MTc2NTZdfQ==
-->