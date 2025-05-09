# Diffuse
# PBR渲染流程加入IBL
## 1 在PBR的Shader中，加入irradianceMap的使用，并且输出颜色实验
此处是作为实验，直接把`irradiance`颜色输出，`irradiance`是我们预积分得到的结果

![输入图片说明](/imgs/2025-04-08/Y3xPs3ic2wj9KwtI.png)

根据公式，预积分得到的`irradiance`还要乘上自身的`color`和`kd`才是真正的环境光的值

```glsl
uniform samplerCube irradianceMap;
void main()
{
	...
	vec3 irradiance = texture(irradianceMap, N).rgb;
	//最终的L = irradiance * color * kd;
	...
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

```cpp
void prepare() {
	...

	//pass 01
	auto geometry = Geometry::createSphere(1.0f);
	material = new PbrMaterial();
	material->mAlbedo = new Texture("assets/textures/pbr/slab_tiles_diff_2k.jpg", 0, GL_SRGB_ALPHA);
	material->mNormal = Texture::createNearestTexture("assets/textures/pbr/slab_tiles_nor_gl_2k.jpg");
	material->mRoughness = Texture::createNearestTexture("assets/textures/pbr/slab_tiles_rough_2k.jpg");
	material->mMetallic = Texture::createNearestTexture("assets/textures/pbr/slab_tiles_arm_2k.jpg");
	material->mIrradianceIndirect = Texture::createExrCubeMap(
		{
			"assets/textures/pbr/IBL/env_0.exr",
			"assets/textures/pbr/IBL/env_1.exr",
			"assets/textures/pbr/IBL/env_2.exr",
			"assets/textures/pbr/IBL/env_3.exr",
			"assets/textures/pbr/IBL/env_4.exr",
			"assets/textures/pbr/IBL/env_5.exr"
		}
	);

	for (int i = 0; i < 5; i++) {
		for (int j = 0; j < 5; j++) {
			auto mesh = new Mesh(geometry, material);
			mesh->setPosition(glm::vec3(i * 2.5, j * 2.5, 0.0f));
			sceneOff->addChild(mesh);
		}
	}

	auto boxGeo = Geometry::createBox(1.0f);
	auto boxMat = new CubeMaterial();
	boxMat->mDiffuse = Texture::createExrTexture("assets/textures/pbr/warm_bar_4k.exr");
	auto boxMesh = new Mesh(boxGeo, boxMat);
	sceneOff->addChild(boxMesh);
	...
	}

}

```
运行输出的结果如下：
因为我们此时在做实验，把`irradiance`作为颜色直接输出了，这个是预积分的值

![输入图片说明](/imgs/2025-04-09/AHVZ12IzTgRTkOx7.png)

## 4 PBR光照当中加入间接光照
我们原本的的`fresnel`项使用的是`H,V`的点乘，这是因为我们的点光源与物体表面只会有一种入射光的情况
```glsl
vec3 H = normalize(L + V);
vec3 F = fresnelSchlick(F0,max(dot(H,V),0.0));
```
但是此时，我们环境光就会有无数条入射光，这时候`H`就不好算了，我们可以直接近似为`N,V`的点乘，`N`是通过法线贴图采样得来的
而且此时，我们算的是环境光照的`diffuse`部分，所以我们可以设计一个新函数`fresnelSchlickRoughness`，加入`Roughness`因素的影响
```glsl
vec3 fresnelSchlickRoughness(vec3 F0,float HdotV, float roughness)
{
    return F0 + (max(vec3(1.0 - roughness), F0) - F0) * pow((1.0 - HdotV), 5.0);
}   

void main()
{
	...
	vec3 irradiance = texture(irradianceMap, N).rgb;

	//最终的L = irradiance * color * kd;
	vec3 kd = 1.0 - fresnelSchlickRoughness(F0,max(dot(N,V),0.0), roughness);
	vec3 ambient = irradiance * albedo * kd;

	//vec3 ambient = vec3(0.1) * albedo;
	Lo += ambient;

	FragColor = vec4(Lo, 1.0);
}
```

# Specular

![输入图片说明](/imgs/2025-04-10/GMdJdnMEeV9D8UJA.png)

![输入图片说明](/imgs/2025-04-10/9x9bXLd1PDeVpCAB.png)

固定`roughness`，简化问题，先假设**观察方向固定，沿着法线方向向内观察**（很重要），以此方向做预积分存储

![输入图片说明](/imgs/2025-04-10/96yaAlwW6BOxhigk.png)

## split sum

![输入图片说明](/imgs/2025-04-19/KjaxNUq2Vhjyq56P.png)

![输入图片说明](/imgs/2025-04-19/u1LsCY5XuuhJmbLf.png)

![输入图片说明](/imgs/2025-04-19/0y5fkrDRCjBusyCS.png)

![输入图片说明](/imgs/2025-04-19/eMijpRUTS2wC4Qw7.png)

把整个积分拆分，对这两个分别做预积分

### 预滤波

![输入图片说明](/imgs/2025-04-19/FVbhFbJp2MvZOFie.png)

这里的权重，其实就是想看哪些光和目光的夹角小，影响就会更大，所以用`L`（入射光）与`N`的点乘，为什么用`N`，因为我们做了一个很重要的假设，**观察方向固定，沿着法线方向向内观察**

![输入图片说明](/imgs/2025-04-19/YoLd77q6m5PlhFCf.png)

### 重要性采样

![输入图片说明](/imgs/2025-04-20/vv9qwPyagni36S9B.png)

![输入图片说明](/imgs/2025-04-20/fQtjxuZCGkovBmmq.png)

![输入图片说明](/imgs/2025-04-20/I48BERwkfF3dBn0U.png)

### 中值向量H的PDF
这个`H`其实就是微平面的法线，我们通过这个`H`可以推断出入射的光线`L`，因为我们的视线是沿着宏平面的`N`看进来的，`H`是视线和入射光线的**中值向量**

![输入图片说明](/imgs/2025-04-20/RLOaX5qWOZnsPT3k.png)

![输入图片说明](/imgs/2025-04-20/UYPD4GOZUCe1yR38.png)

![输入图片说明](/imgs/2025-04-20/epPNwRaIi1xbD6eU.png)

![输入图片说明](/imgs/2025-04-20/ewktas5YOh6tsadu.png)

![输入图片说明](/imgs/2025-04-20/sgZa9ozpm6AYNYCp.png)

### 低差异序列算法

左边的不好，有的地方密集有的地方稀疏，我们希望得到右边的效果

![输入图片说明](/imgs/2025-04-20/NiVDeXgrIKCUM79z.png)

![输入图片说明](/imgs/2025-04-20/1YCPfDvTAI0f9mLd.png)

![输入图片说明](/imgs/2025-04-20/CumLVjPJvxzh1Uqs.png)

以`8bits`举例
`bits = (bits << 4u) | (bits >> 4u)`
把`bits`分别左移和右移四位
就会得到`efgh 0000`和`0000 abcd`，最后取一个或
就可以得到`efgh abcd`

![输入图片说明](/imgs/2025-04-20/z5oP5MHb5TFPpoOK.png)

最后返回的是一个二维向量，就是我们之前推导的，`x`就是普通的`i / N`，`y`通过设计的方法进行计算，
这个`2.328306`一长串是用来做归一化的

`0x00FF00FF`理解为蒙版，我们现在要做把两小段的左边变到分别的右边，利用蒙版把不需要的部分置为`0`，然后就可以进行右移了

![输入图片说明](/imgs/2025-04-20/02zgmR9qtLWBn11B.png)

## 流程梳理

![输入图片说明](/imgs/2025-04-20/SFBvYCPxHL9E7jVY.png)

![输入图片说明](/imgs/2025-04-20/RzXVhnwpFRuOA0z5.png)

![输入图片说明](/imgs/2025-04-20/fRQzpuMH69b1yxMk.png)

## 预积分与MipMap

![输入图片说明](/imgs/2025-04-20/4m0xZHrWJPzdqagR.png)

![输入图片说明](/imgs/2025-04-20/tPrInOf3Ink5EEAE.png)

使用`MipMap`可以自动做插值

![输入图片说明](/imgs/2025-04-20/r4Rjre4vmieyXDO6.png)
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTU2NTEzMDkxMCwtMTUyNDA1OTE5NSwtMT
gxMjI4MTA3NiwxMTc3NDIxMjMyLDgwODAxODg4MywxODg2Nzkz
MTc4LDIxMDU3ODg1MTYsODQ4ODAyOTk0LC0xNzEwMTU5NzM3LD
E3MDI0ODIzODMsMjEyOTEwNDM4MCw0NTE1NTgzNDIsODk2Mzkw
NTUsLTE3Mjg4MzYxMDUsLTEzOTM2ODg2NTUsLTE5ODQ5NzI3Mz
gsMTk5NzE3NzIxOCwzODMwNjk2NTAsLTIwNDIzNjA0OTksLTI2
OTYzMTU0MF19
-->