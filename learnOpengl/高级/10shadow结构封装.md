![输入图片说明](/imgs/2025-02-26/aL4BSM1rT8dVSror.png)

这里设计的几个参数，解释几个看起来比较抽象的
当场景中的灯多起来后，就需要多个`shadowMap`，所以`camera, renderTarget`就要设计在`shadow`类下，每一个不同阴影都要有自己的`camera`和`renderTarget`
而这个`setRenderTargetSize`方法，在前面加上`virtual`，后面加上`=0`，说明这是一个纯虚函数，需要子类强制进行实现，
以前我们是直接写死在`renderer.cpp`中，如下：
`mShadowFBO = Framebuffer::createShadowFbo(2048, 2048);`
大小为`2048*2048`，而我们以后肯定会有不同`shadowMap`大小的定制需求，所以子类需要强制进行重写方法

![输入图片说明](/imgs/2025-02-26/DsjfF0lOja219sUp.png)

![输入图片说明](/imgs/2025-02-26/mfl7Gdi2fZhSKIvj.png)

# 实现
## 1.加入Shadow父类，并且派生DirectionalLightShadow子类
### 1.1 注意setRenderTargetSize的实现
### 1.2 DirectionalLightShadow中，加入getLightMatrix函数
## 2.在Light父类中，加入Shadow类型对象，并且在平行光中对其进行初始化（其余暂时不管）	
## 3.RenderShadowMap中更改FBO与lightMatrix获取方式
## 4.RenderObject中，更改ShadowMap以及其他系列参数的更新
## 5.IMGUI中修改对bias、pcfRadius、tightness、renderTarget大小的调整
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE4MjgwMjAzMDEsLTEwMzA2Mjc3NzYsOT
I1MjI3NjM3LDQ2Njc1NjUxNF19
-->