# PBR渲染流程加入IBL
## 1 在PBR的Shader中，加入irradianceMap的使用，并且输出颜色实验
此处是作为实验，直接把`irradiance`颜色输出，
```glsl
uniform samplerCube irradianceMap;
void main()
{
	...
	vec3 irradiance = texture(irradianceMap, N).rgb;
	FragColor = vec4(irradiance, 1.0);
}
```
## 2 在pbrMaterial中，加入irradianceIndirect纹理（CubeMap，HDR纹理）
```cpp
...
public:
	Texture* mAlbedo{ nullptr };	//双线性插值
	Texture* mRoughness{ nullptr };	//最临近插值
	Texture* mNormal{ nullptr };//最临近插值
	Texture* mMetallic{ nullptr };//最临近插值
	Texture* mIrradianceIndirect{ nullptr };//环境贴图IBL
};
```
相应的要去`renderer.cpp`中的渲染函数加入`irradianceMap`贴图的传输
```cpp
case MaterialType::PbrMaterial: {
	...
	shader->setInt("irradianceMap", 4);
	pbrMat->mIrradianceIndirect->setUnit(4);
	pbrMat->mIrradianceIndirect->bind();
	...
}
	break;
```
## 3 构造场景，加入天空盒
## 4 PBR光照当中加入间接光照
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTExOTY5NDEzOTAsMTYxMTk4Nzk0N119
-->