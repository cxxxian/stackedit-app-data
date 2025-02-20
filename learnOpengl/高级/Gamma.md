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
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTg3NzQzNTEyNSwtNTc1ODk3NDMsLTMyMz
M0MTA5MCwtMjc3Njk1OTI4LC0zMTA1MTg1NjEsMTYxMDQ5MDAy
OSwtMTE3NjMzNDQ4NF19
-->