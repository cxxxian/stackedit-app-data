![输入图片说明](/imgs/2025-02-26/aL4BSM1rT8dVSror.png)

这里设计的几个参数，解释几个看起来比较抽象的
当场景中的灯多起来后，就需要多个`shadowMap`，所以`camera, renderTarget`就要设计在`shadow`类下，每一个不同阴影都要有自己的`camera`和`renderTarget`
而这个`setRenderTargetSize`方法，在前面加上`virtual`，后面加上`=0`，说明这是一个纯虚函数，需要子类强制进行实现，
以前我们是直接写死在`renderer.cpp`中，如下：
`mShadowFBO = Framebuffer::createShadowFbo(2048, 2048);`
大小为`2048*2048`，而我们以后肯定会有不同`shadowMap`大小的定制需求，所以子类需要强制进行重写方法

![输入图片说明](/imgs/2025-02-26/DsjfF0lOja219sUp.png)

![输入图片说明](/imgs/2025-02-26/mfl7Gdi2fZhSKIvj.png)
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTEwMzA2Mjc3NzYsOTI1MjI3NjM3LDQ2Nj
c1NjUxNF19
-->