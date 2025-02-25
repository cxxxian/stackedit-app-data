# 概念

![输入图片说明](/imgs/2025-02-25/bt5KiPgBKRaSixIZ.png)

从光源相机视角：
`shadowMap`由于可能分辨率不够，玩家视角的三个像素对应到`shadowMap`可能是同一个像素，（也可能玩家视角离得近，自然看到的像素就更多）
但是`shadowMap`肯定没办法记录那么多个，所以会选取中间那个进行存储，
那么从玩家视角下，很明显深度`d0 > d1 > d2`，所以`d0`会被误认为遮挡

![输入图片说明](/imgs/2025-02-25/ah8tjNTdtm30xfea.png)

所以`bias`不是一个固定值，可以暴露出来给美术人员调整，不同场景需要不同的`bias`

# 代码实现
## 1.shader中加入uniform变量bias，并且在阴影判断中使用
用`selfDepth - bias`，就可以把观察到的像素向光源推进`bias`个距离，消除子遮挡现象
```glsl
uniform float bias;
...
float calculateShadow(){
	...
	float shadow = (selfDepth - bias) > closestDepth? 1.0:0.0;
	return shadow;
}
```
相应的我们要在`cpu`端设计参数并传输，
在`phongShadowMaterial.h`设计参数`mBias`
```cpp
public:
	...
	float		mBias{ 0.0f };
```
然后就到`renderer.cpp`中的`renderObject`找到对应`case`进行传输即可
case MaterialType::PhongShadowMaterial: {
	PhongShadowMaterial* phongShadowMat = (PhongShadowMaterial*)material;

	shader->setInt("sampler", 0);
	phongShadowMat->mDiffuse->bind();

	shader->setInt("shadowMapSampler", 1);
	mShadowFBO->mDepthAttachment->setUnit(1);
	mShadowFBO->mDepthAttachment->bind();


	shader->setMatrix4x4("lightMatrix", getLightMatrix(dirLight));

	//mvp
	shader->setMatrix4x4("modelMatrix", mesh->getModelMatrix());
	shader->setMatrix4x4("viewMatrix", camera->getViewMatrix());
	shader->setMatrix4x4("projectionMatrix", camera->getProjectionMatrix());

	auto normalMatrix = glm::mat3(glm::transpose(glm::inverse(mesh->getModelMatrix())));
	shader->setMatrix3x3("normalMatrix", normalMatrix);

	//光源参数的uniform更新
	//directionalLight 的更新
	shader->setVector3("directionalLight.color", dirLight->mColor);
	shader->setVector3("directionalLight.direction", dirLight->getDirection());
	shader->setFloat("directionalLight.specularIntensity", dirLight->mSpecularIntensity);
	shader->setFloat("directionalLight.intensity", dirLight->mIntensity);

	shader->setFloat("shiness", phongShadowMat->mShiness);

	shader->setVector3("ambientColor", ambLight->mColor);

	//相机信息更新
	shader->setVector3("cameraPosition", camera->mPosition);

	//透明度
	shader->setFloat("opacity", material->mOpacity);

	shader->setFloat("bias", phongShadowMat->mBias);

}
	break;
## 2.在IMGUI中加入对bias的调节
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE5Mzg5NjgzODYsLTE0MTkwMjU4OTAsLT
EyMjMxODc5NjIsLTIwODg3NDY2MTJdfQ==
-->