# 准备工作
## 1.改造Light类，继承Object（并为Object增加getDirection函数）
## 2.更改使用光源的代码们

# ShadowMap
## shader制作
### 1.创建shadow.vert/frag，渲染阴影贴图专用
### 2.在Renderer中创建shadowShader，用于做ShadowMap渲染

## 渲染目标
### 1.在Texture中增加创建DepthAttachment的创建函数
### 2.在FrameBuffer中增加创建ShadowFBO的创建函数
### 3.在Renderer中创建ShadowFBO，用于做阴影ShadowMap的渲染目标（RenderTarget）

## 渲染器修改
在`Renderer`中加入`RenderShadowMap`函数，在真正渲染物体之前，先把`ShadowMap`做出来
### 注意1：
做好排除工作，ScreenMaterial的物体不参与ShadowPass渲染
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTMzODIxMDYwMl19
-->