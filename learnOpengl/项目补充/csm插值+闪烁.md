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
主要原因就是我们现在做的是`CSM`，这个包围盒我们是利用玩家视角的摄像机从`NDC`去反推出世界坐标的！所以

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
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTg2MDk2NjgyOSwtMTg0MTk1NTE5OF19
-->