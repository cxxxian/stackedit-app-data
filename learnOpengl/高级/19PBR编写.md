![输入图片说明](/imgs/2025-03-09/H5fQTdOQ9h2nCRVH.png)

# 准备工作
## 1 创建pbr的Material以及shader
`pbrMaterial.h`如下
```cpp
#pragma once
#include "../material.h"
#include "../../texture.h"

class PbrMaterial :public Material {
public:
	PbrMaterial();
	~PbrMaterial();
	...
};
```
`pbrMaterial.cpp`如下，以及去到`material.h`中添加对应枚举类
```cpp
#include "pbrMaterial.h"

PbrMaterial::PbrMaterial() {
	mType = MaterialType::PbrMaterial;
}

PbrMaterial::~PbrMaterial() {

}
```
```
```
## 2 pbr材质解析加入管线中
## 3 编写D G F三大函数

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTEzOTU5MDY4NTIsLTE3Mzc1OTU1NTYsLT
IwMTMyNDgzNjUsMTEyOTY4MjUxNCwtMjA4ODc0NjYxMl19
-->