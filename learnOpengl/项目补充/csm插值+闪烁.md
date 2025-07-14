# 插值
## 改动的部分（加入层间平滑）

```glsl
// ---------- 新增 Uniform ----------
uniform float cascadeBlendRange;     // 世界空间中用于层间混合的过渡带宽度

// ---------- 新增函数：返回两个层索引及权重 ----------
void getLayersAndWeight(float z, out int layer0, out int layer1, out float w){
    // 找到主层（layer0）
    layer0 = 0;
    for(int i = 0; i < csmLayerCount; ++i){
        if(z < csmLayers[i]){ layer0 = i - 1; break; }
    }
    layer0 = clamp(layer0, 0, csmLayerCount - 1);
    layer1 = min(layer0 + 1, csmLayerCount - 1);

    // 计算混合权重 w∈[0,1]
    float farBound = csmLayers[layer1];
    float blendStart = farBound - cascadeBlendRange;
    w = (z > blendStart && layer0 != layer1)
        ? smoothstep(0.0, 1.0, (z - blendStart) / cascadeBlendRange)
        : 0.0;
}

// ---------- 替换原来的 csm(...) ----------
float csm(vec3 positionWorldSpace, vec3 normal, vec3 lightDir, float pcfRadius){
    // 1. 相机空间深度
    vec3 posCam = (viewMatrix * vec4(positionWorldSpace,1.0)).xyz;
    float z = -posCam.z;

    // 2. 取得两个层及权重
    int layer0, layer1; float w;
    getLayersAndWeight(z, layer0, layer1, w);

    // 3. 主层采样
    vec4 clip0 = lightMatrices[layer0] * vec4(positionWorldSpace,1.0);
    float s0 = pcf(clip0, layer0, normal, lightDir, pcfRadius);

    // 4. 若在过渡带，再采样远层并混合
    if(w == 0.0 || layer0 == layer1) return s0;

    vec4 clip1 = lightMatrices[layer1] * vec4(positionWorldSpace,1.0);
    float s1 = pcf(clip1, layer1, normal, lightDir, pcfRadius);

    return mix(s0, s1, w);   // 平滑过渡
}

```

### 使用说明

1.  **`cascadeBlendRange`** 建议占该 cascade 深度 5 %–10 %（单位=世界空间）；  
    例：`cascadeBlendRange = 5.0`（或随层级比例调节）。
    
2.  仍然调用原来的 `shadow = csm(...)`，其它逻辑保持不变。
    

这样，当像素落入相邻两张 shadow‑map 的重叠区时，shader 会自动再采样下一层并用 `smoothstep` 做权重插值，纹理分辨率骤变带来的亮/暗跳变即可消除。

## 总结
## 插值怎么做？

我们选择“**世界空间 z 深度**”作为插值因子（也可以是相机空间 z）：

### 插值公式：

```glsl
finalShadow = mix(shadow0, shadow1, w);
```

其中：

-   `shadow0`：当前 cascade（近层）的阴影值
    
-   `shadow1`：下一层 cascade（远层）的阴影值
    
-   `w`：权重，取值 [0,1]，表示像素在交界处的位置

用大白话讲其实跟贴图没有任何关系，插值步骤体现在最后计算出来的`shadow`值，在两个相邻层级之间，对不同两个`shadowMap`得出的`shadow`值做插值得出一个平滑过滤

# 闪烁
## 一句话概括：

> 当摄像机或光源轻微移动时，**阴影纹理中的深度映射 texel 没有与世界空间固定对齐**，导致阴影边缘在屏幕上“跳来跳去”。
> 
## 场景设定：

以 **定向光+CSM** 为例，你为每个 cascade 创建了一个 **正交投影矩阵**：

```glsl
lightMatrix = proj * lightView;
```

这个正交矩阵定义了当前 `shadow map` 在光源空间中能“看到”的区域：

-   比如：`x ∈ [-50, 50], y ∈ [-50, 50]`，宽高共 `100` 个世界单位；
    
-   你把它渲染到一个 `2048×2048` 的 `shadow map` 中；
    
-   所以：**每个 texel 大约对应世界空间中的 100/2048 ≈ 0.0488 个单位**。
    

----------

## 问题来了

你会发现，**只要摄像机稍微动一动，lightMatrix 的投影中心就会移动一点点**，比如 `0.01` 的单位，但 `shadow map` 是离散的 `texel`，它无法“半格”移动，于是采样位置会跳跃：

```cpp
摄像机移动前：
     世界空间点 P → 落到 shadow map 第 512×512 格

摄像机微动：
     世界空间点 P → 落到 shadow map 第 513×512 格
```

这时即使阴影在真实世界中没有动，**屏幕上的阴影就跳了一格**，你看到的就是**闪烁或游移现象**。

再解释一下为什么`lightMatrix`会和摄像机扯上关系，明明`lightMatrix`只和光源有关啊。
主要原因就是我们现在做的是`CSM`，这个包围盒我们是利用玩家视角的摄像机从`NDC`去反推出世界坐标的！所以这里就和玩家视角摄像机扯上关系了


----------

## 怎么解决？

我们要做的是：

> 让 shadow map 的纹理坐标 **固定在世界空间的整数 texel 格上**，也就是：**锁定投影中心，使其始终对齐到 texel 大小的整数倍**。

----------

### 计算步骤（核心）

你可以在构造 lightView → lightProj 矩阵时，加一步 **对中心坐标进行 texel 级对齐**。

假设：

-   shadow map 分辨率：`resolution = 2048`
    
-   你已经得出当前视锥在光源空间下的正交包围盒大小为 `width, height`（比如 100）
    
-   那么一个 texel 对应的世界单位大小：

```glsl
float worldTexelSize = width / resolution;
```

然后你对包围盒中心 `vec3 centerWS` 做对齐：

```glsl
vec3 centerLS = lightView * vec4(centerWS, 1.0);  // 转到 light space

centerLS.xy = worldTexelSize * floor(centerLS.xy / worldTexelSize); // 向下对齐
```


然后用对齐后的中心 `centerLS` 构造新的正交投影盒 → 投影矩阵。

## 让我们用一个例子更清晰的弄懂

### 简化问题版本：

你有一个物体点 P，在世界坐标系中是固定的，比如：

```
P_world = (5.0, 0.0, 5.0)     // 地面上的某点
```

你摄像机轻轻动了一下，比如从 `(0.0, 5.0, -10.0) → (0.01, 5.0, -10.0)`，想让它尽可能不影响阴影。

问题是，这个点 P 最终通过以下流程映射到 shadow map：

```
P_uv = (lightProj * lightView) * P_world;
```

lightView 是光源固定方向决定的，但 **lightProj 是根据摄像机包围盒裁出来的**，  
所以只要摄像机轻动一点点 → 包围盒中心也轻微改变 → lightProj 随之微调。

于是：

-   没有对齐时：`lightMatrix` 每帧都略有不同 → P_uv 每帧落在不同 texel 上 → 阴影像素闪烁。
    
-   对齐之后：我们人为强制 `lightProj` 中心吸附到“texel网格点”，只要偏移 **不超过一个 texel 宽**，投影中心就不动 → `lightMatrix` 不动 → 阴影稳定 ✅。
    

----------

### 🔍 我们现在更细地看看两个版本：

----------

### ❌ **不对齐的版本：**

假设你第一帧包围盒中心落在：

```cpp
centerLS.x = 1.023
```

第二帧因为摄像机稍微动了一点，变成：

```cpp
centerLS.x = 1.026
```

虽然只差 0.003，但这会让整个投影矩阵的中心变成不同位置 →  
P_world 在 shadow map 上的 `uv` 坐标也跟着微变，例如从 (0.502, 0.497) → (0.505, 0.499)  
**这样一变化，最终采样的 texel 可能就变了**，即使结果看起来差不多，也会导致：

> 屏幕上的像素突然从“亮”变“暗” → 抖动 ❌

----------

### ✅ **对齐后的版本：**

我们做了这个操作：

```cpp
centerLS.x = texelSize * floor(centerLS.x / texelSize);
```

举例：

-   texelSize = 0.0488
    
-   `centerLS.x = 1.023` → floor(1.023 / 0.0488) = 20 → 对齐到 20 × 0.0488 = **0.976**
    
-   第二帧：`centerLS.x = 1.026` → 仍然 floor(1.026 / 0.0488) = 20 → **仍然对齐到 0.976**
    

虽然中心值实际在变，但我们把它吸附到了固定位置，**只要没有跨过一个完整 texel（0.0488）就不变**。

于是：

> lightProj 不变 → lightMatrix 不变 → 同一个 P_world 映射到相同的 `uv` → 阴影采样不跳 → 屏幕像素稳定 ✅

----------

### 🧠 更形象的比喻

把 shadow map 看成一张纸上的网格（格子大小 = texelSize）

-   摄像机动一动，等于你拖动整张纸的“可视框”
    
-   没对齐时：纸跟着摄像机连续滑动，格子边缘在你眼中在“晃”
    
-   对齐后：你只允许视窗每次滑一个格子，格子就稳定地对齐视窗，不会在视觉上跳来跳去
    

----------

### ✅ 总结一句话：

> **不对齐时：每帧的 lightMatrix 总有微小变化，导致阴影采样落在不同 texel，产生闪烁。**  
> **对齐后：只要摄像机没有跨过一个 texel，lightMatrix 就锁定不变，阴影采样点固定，因此画面稳定。**

# 解析`centerLS.x = texelSize * floor(centerLS.x / texelSize);`

为什么要这样算
### `centerLS.x` 是什么？

```vec3 centerLS = (lightView * vec4(centerWS, 1.0)).xyz;`

-   `centerWS` 是你当前主摄像机视锥包围盒的中心，单位是“世界坐标”
    
-   `lightView` 是光源的视图矩阵，作用是把“世界坐标”变成 **光源空间（Light Space）坐标**
    
-   所以 `centerLS` 仍然是“以米为单位的坐标值”，只不过是光源视角下的位置
    

✅ 它是**光空间下的世界坐标位置**，不是纹理坐标。
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE3ODM3OTc4NTgsLTY3Njc0NDYwNSwtMT
kzNDQzODgyMiwtMTg0MTk1NTE5OF19
-->