![输入图片说明](/imgs/2025-02-26/aL4BSM1rT8dVSror.png)

这里设计的几个参数，解释几个看起来比较抽象的
当场景中的灯多起来后，就需要多个`shadowMap`，所以`camera, renderTarget`就要设计在`shadow`类下，每一个不同阴影都要有自己的`camera`和`renderTarget`
而这个`setRenderTargetSize`方法，在前面加上`virtual`，后面加上`=0`，说明这是一个纯虚函数，需要子类强制进行实现，
以前我们是直接写死
`mShadowFBO = Framebuffer::createShadowFbo(2048, 2048);`
<!--stackedit_data:
eyJoaXN0b3J5IjpbMjMzNDc2MjY3XX0=
-->