![输入图片说明](/imgs/2025-02-20/A1yXOH1zq4ZvfL4F.png)

通过人眼会觉得上面的灰度图比较均匀，但实际上下面的灰度图才是真正均匀变换的

![输入图片说明](/imgs/2025-02-20/9fHjOACBX7sbBFwe.png)

![输入图片说明](/imgs/2025-02-20/ek3VoXmYMyB8IFb0.png)

![输入图片说明](/imgs/2025-02-20/cHqbi1VatlFS3Y1Y.png)

![输入图片说明](/imgs/2025-02-20/aTA0oXUxrFtmYlsW.png)

这里的`color-srgb`不是减法，时`srgb`空间下的`color`值的意思

![输入图片说明](/imgs/2025-02-20/2XzOzac53WpulaD2.png)

![输入图片说明](/imgs/2025-02-20/Zylj3ilFAZ2iE6G6.png)

![输入图片说明](/imgs/2025-02-20/7DeRo2yo5ijHrFab.png)

这张图的意思是，我们从`ps`导出图片时，图片会从`32`位精度转化为`8`位精度，即从`0~1`的`float`转为`255`，
但是我们将图片读取到`OpenGL`中时，又会被转回去`0~1`的`float`类型，因为我们在`OpenGL`需要对图片做一些计算，转化为`0~1`比较方便
此时就会发现丢失精度，比如：
`0.218`和`0.22`都会被转为`56`，`56`再转回来两个就都变成`0.22`了

![输入图片说明](/imgs/2025-02-20/uDqFLhbQl2jOAusP.png)

![输入图片说明](/imgs/2025-02-20/ZGlplixmTgm5KvWz.png)

我们在导出图片时，先做运算转化为`sRGB`空间，这样`32`转化为`8`位存储空间的时候，暗部就不至于丢掉太多细节精度，然后在我们将图片读入`OpenGL`时，再做相应的运算把`sRGB`转化回`RGB`空间

![输入图片说明](/imgs/2025-02-20/I9uhYuLekoPbj64p.png)

![输入图片说明](/imgs/2025-02-20/b45R57UWlAkkR7IK.png)

![输入图片说明](/imgs/2025-02-20/8OoTVsUTofkFySlj.png)

![输入图片说明](/imgs/2025-02-20/I65fa7Oravmm7mZK.png)

![输入图片说明](/imgs/2025-02-20/9PTHY9kv6gaF8czf.png)

![输入图片说明](/imgs/2025-02-20/gJwSWBlCBj8NfBrL.png)

![输入图片说明](/imgs/2025-02-20/Jd8EoZQYJj8fNJPv.png)
<!--stackedit_data:
eyJoaXN0b3J5IjpbNDE1MjMwNTksLTM5MzE3ODA3Miw0Nzc5ND
A4NDcsLTY4NjIwMTc1NCwtNTM1OTE4OTgyLC05ODIzNDIwMjMs
LTU3NTg5NzQzLC0zMjMzNDEwOTAsLTI3NzY5NTkyOCwtMzEwNT
E4NTYxLDE2MTA0OTAwMjksLTExNzYzMzQ0ODRdfQ==
-->