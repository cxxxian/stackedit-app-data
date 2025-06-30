一、选择 `Deferred Rendering` 的典型场景
1. 核心优势
超多动态光源支持：
延迟渲染的核心原理是将 几何信息（G-Buffer） 与 光照计算分离
光照阶段仅依赖 G-Buffer 数据，与场景复杂度无关
典型场景：城市夜景（数千动态光源）、科幻场景（大量发光物体）
2. 技术实现关键

```cpp
// G-Buffer 示例结构（OpenGL）
GLuint gBuffer;
glGenFramebuffers(1, &gBuffer);
glBindFramebuffer(GL_FRAMEBUFFER, gBuffer);

// 颜色+法线+位置+材质 多渲染目标（MRT）
GLuint gPosition, gNormal, gAlbedo, gSpecular;
glGenTextures(1, &gPosition);
glBindTexture(GL_TEXTURE_2D, gPosition);
glTexImage2D(... GL_RGBA16F ...); // 高精度存储位置
// 类似创建其他附件...
glFramebufferTexture2D(GL_FRAMEBUFFER, GL_COLOR_ATTACHMENT0, GL_TEXTURE_2D, gPosition, 0);
// 绑定其他附件到 GL_COLOR_ATTACHMENT1~3

// 深度渲染缓冲
GLuint rboDepth;
glGenRenderbuffers(1, &rboDepth);
glBindRenderbuffer(GL_RENDERBUFFER, rboDepth);
glRenderbufferStorage(GL_RENDERBUFFER, GL_DEPTH_COMPONENT, width, height);
glFramebufferRenderbuffer(GL_FRAMEBUFFER, GL_DEPTH_ATTACHMENT, GL_RENDERBUFFER, rboDepth);
```
3. 适用平台
`PC`/主机等高性能平台：
`G-Buffer` 的带宽消耗在高性能 `GPU` 上可接受
典型分辨率：`4K` 下 `G-Buffer` 总带宽 ≈ `500MB`（`RGBA16F x4`）
需要复杂后处理：
`SSAO`、`SSR` 等屏幕空间效果直接读取 `G-Buffer`

透明物体处理：
需要额外 Forward Pass 渲染透明物体
5. 典型案例
《赛博朋克 2077》：
密集霓虹灯、车灯、全动态光照
使用改进版 `Deferred + Ray Tracing` 混合管线
二、选择 `Forward+` 的典型场景
1. 核心优势
带宽效率：
不需要存储完整的 `G-Buffer`
移动端等带宽敏感平台优势明显
材质多样性支持：
复杂材质（毛发、皮肤 `SSS`）可直接在着色器实现
无 `G-Buffer` 精度限制（如法线压缩失真）
2. 技术实现关键
```cpp
// 核心：分块光源列表（Tile-Based Light Lists）
uniform sampler2D depthTexture; // 深度缓冲区
uniform mat4 invProjection;      // 逆投影矩阵

// 计算屏幕空间分块（例如 16x16 像素块）
for (int tileY = 0; tileY < numTilesY; tileY++) {
    for (int tileX = 0; tileX < numTilesX; tileX++) {
        // 计算分块在视图空间的 AABB 包围盒
        vec3 minBounds = vec3(INFINITY);
        vec3 maxBounds = vec3(-INFINITY);
        for (int y = 0; y < 16; y++) {
            for (int x = 0; x < 16; x++) {
                vec2 uv = vec2((tileX*16+x)/screenWidth, (tileY*16+y)/screenHeight);
                float depth = texture(depthTexture, uv).r;
                vec3 viewPos = ViewSpaceFromDepth(uv, depth, invProjection);
                minBounds = min(minBounds, viewPos);
                maxBounds = max(maxBounds, viewPos);
            }
        }
        // 收集影响该分块的光源索引
        lightIndices[tileX][tileY] = FindAffectingLights(minBounds, maxBounds);
    }
}
```
3. 适用平台
移动端/`VR` 设备：
`Adreno/Mali GPU` 的 `Tile-Based` 架构天然适配

需要 `MSAA` 的场景：
`Forward` 管线天然支持硬件 `MSAA`
`Deferred` 的 `MSAA` 需要多重采样 `G-Buffer`（成本极高）
4. 性能瓶颈
光源数量限制：
每个分块的光源数受限于 `GPU` 寄存器（通常 `≤ 256`）
动态分支过多可能降低 `SIMD` 利用率
复杂光源类型：
投射阴影的聚光灯处理不如 `Deferred` 高效

![输入图片说明](/imgs/2025-04-14/yQECMKOfXvo9d78J.png)


# Forward+ 的原理

Forward+（也称为 Clustered Forward Shading 或 Tiled Forward Shading）是一种先进的光照技术，旨在优化多光源场景的渲染性能。它是传统 Forward Rendering 的扩展，通过在光照计算之前引入一个基于计算着色器（Compute Shader）的光源剔除阶段，减少了每个像素需要处理的光源数量。

#### 1. **光源剔除（Light Culling）**

Forward+ 的核心在于光源剔除阶段。在这个阶段，屏幕被划分为多个小的区域（通常称为 Tile 或 Cluster），每个区域的大小通常是 16x16 或 32x32 像素。对于每个 Tile，计算其对应的视锥体（Bounding Box），然后将场景中的光源与这些视锥体进行相交测试，以确定哪些光源对当前 Tile 有贡献。

#### 2. **光源列表生成**

在光源剔除阶段，会生成两个光源索引列表：

-   **不透明光源索引列表**：用于不透明几何体的光照计算。
    
-   **透明光源索引列表**：用于透明几何体的光照计算。
    

这些列表存储了影响每个 Tile 的光源索引，从而在后续的光照计算中，每个像素只需要考虑其所在 Tile 的光源列表，而不是场景中的所有光源。

#### 3. **前向渲染（Forward Rendering）**

在光源剔除阶段之后，使用标准的前向渲染流程来对场景中的物体进行着色。与传统的前向渲染不同，Forward+ 在计算每个像素的光照时，只会遍历当前像素所在 Tile 的光源列表，而不是场景中的所有光源。

#### 4. **性能优化**

Forward+ 的主要优势在于减少了光照计算的复杂度。通过将光源分组到 Tile 中，每个像素只需要处理少量的光源，而不是整个场景中的所有光源。这显著减少了光照计算的开销，提高了渲染性能。

#### 5. **应用场景**

Forward+ 特别适用于场景中有大量动态光源的情况，如粒子系统、动态灯光效果等。它能够有效地处理这些光源，而不会显著降低性能。

### 总结

Forward+ 是一种结合了传统前向渲染和基于 Tile 的光源剔除技术的渲染方法。它通过在光照计算之前引入光源剔除阶段，减少了每个像素需要处理的光源数量，从而优化了多光源场景的渲染性能。这种方法在处理大量动态光源时表现出色，同时支持不透明和透明物体的渲染

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE3OTI1MTMzMzAsNjMyNjMyNzE3LDU4Mz
AxNDEwMSw0NjExNDkxNjJdfQ==
-->