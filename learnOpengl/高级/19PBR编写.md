![输入图片说明](/imgs/2025-03-09/H5fQTdOQ9h2nCRVH.png)

# 准备工作
## 1 创建pbr的Material以及shader
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
``
## 2 pbr材质解析加入管线中
## 3 编写D G F三大函数

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTIxMTA4NDMxMCwtMjAxMzI0ODM2NSwxMT
I5NjgyNTE0LC0yMDg4NzQ2NjEyXX0=
-->