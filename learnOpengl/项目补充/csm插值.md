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


<!--stackedit_data:
eyJoaXN0b3J5IjpbNjY4NjMxNzgxXX0=
-->