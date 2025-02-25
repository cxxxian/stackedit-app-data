# 准备工作
## 1.改造Light类，继承Object（并为Object增加getDirection函数）
因为之前我们说过，`light`其实也是有位置的，所以我们可以把`Object`做为`Light`的父类（当初设计`Object`类就有位置等参数），
继承后就可以利用`Object`的`getModelMatrix`得到`M`矩阵，而上节课说到的，`M`的逆矩阵就是光源摄像机的`viewMatrix`
然后我们原本的`directionLight`有一个参数是：
```cpp
public:
	glm::vec3 mDirection{-1.0};
```
但现在光源既然继承于`Object`，那它就可以进行旋转什么的，所以我们要为`Object`增加`getDirection`函数，因为此时的`mDirection`可能会因为旋转什么的发生变化

开始修改代码：
将`Light`继承于`Object`，这样所有的光源就都继承于`Object`了，因为`light.h`是所有光源的父类
```cpp
#pragma once
#include "../core.h"
#include "../object.h"
class Light: public Object {
	...
};
```
然后去把所有光源有关于方向的可以直接删除了
检查过后发现只有`directionLight`和`spotLight`有
分别是
```cpp
public:
	glm::vec3 mDirection{-1.0};
```
和
```cpp
glm::vec3	mTargetDirection{ -1.0f };
```

现在我们开始设计`geiDirection`方法

![输入图片说明](/imgs/2024-11-01/rKYoiuZ2ttQqLKy0.png)

我们先回忆一下先前设计的旋转矩阵，第一列是`right`轴，第二列是`up`轴，第三列是`front`轴（取`-f`是因为我们的向前对应的是`z`方向），分别对应`xyz`

因为我们要得到光照的方向，所以只要关注旋转矩阵的`front`方向即可，代表向前的向量
有了这个回忆，我们就可以开始设计了
在`object.h`声明并在`object.cpp`实现：
```cpp
glm::vec3 Object::getDirection() const {
	auto modelMatrix = glm::mat3(getModelMatrix());
	auto dir = glm::normalize(-modelMatrix[2]) ;

	return dir;
}
```
解释一下，可以利用上面那张图，`getModelMatrix()`此时得到的是一个四维矩阵左上角的`3*3`矩阵是旋转矩阵，第四列的前三个代表平移，而第四行是我们为了齐次进行的补全
所以我们通过`glm::mat3(getModelMatrix())`就可以获得旋转矩阵
而此时我们只需要`front`向量，所以`modelMatrix[2]`就可以取得`modelMatrix`的第三列即`front`，最后还需要加上一个负号，因为此时是朝向`z`轴的，但是`-z`轴才是我们的前方

## 2.更改使用光源的代码们
由于我们删除了`directionLight`和`spotLight`的方向，肯定会有很多报错
慢慢来改
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
做好排除工作，`ScreenMaterial`的物体不参与`ShadowPass`渲染，若是`PostProcessPass`则不进行`RenderShadowMap`的操作（防止污染`ShadowMap`）

### 注意2：
做好备份工作，先前的`fbo`，先前的`viewport`等参数，都需要做备份与恢复
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTMyMTAwNjkxMCw3Nzk1Mjc2MzcsMzEzMT
EyNDQzLC0xODYwMTY5NjExLC0yMTg3NzcxMzUsLTMzODIxMDYw
Ml19
-->