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


### Early Z、Depth PrePass、HiZ 和 Late Z 的概念及用途

这些技术都属于渲染管线中的深度测试和优化技术，旨在减少不必要的计算和内存访问，提高渲染性能。它们可以统称为**深度测试优化技术**或**遮挡剔除技术**。

#### 1. **Early Z（提前深度测试）**

**概念**： Early Z 是一种优化技术，通过在光栅化阶段和片元着色器阶段之间增加一个额外的深度测试步骤，提前丢弃被遮挡的片元，从而减少不必要的片元计算。

**用途**：

-   **减少 OverDraw**：通过提前丢弃被遮挡的片元，减少不必要的片元着色器计算，从而提高渲染性能。
    
-   **优化渲染顺序**：在渲染不透明物体时，从近到远的顺序渲染可以最大化 Early Z 的优化效果。
    

**失效情况**：

-   开启 Alpha Test 或 clip/discard 等手动丢弃片元操作。
    
-   手动修改 GPU 插值得到的深度。
    
-   开启 Alpha Blend。
    
-   关闭深度测试 Depth Test。
    

#### 2. **Depth PrePass（深度预通道）**

**概念**： Depth PrePass 是一种优化技术，通过两个渲染阶段来减少 OverDraw。在第一个阶段（Z-Prepass），只渲染物体的深度信息，不计算颜色。在第二个阶段，关闭深度写入，将深度比较函数设置为相等，进行正常的着色计算。

**用途**：

-   **减少 OverDraw**：通过提前生成深度缓冲区，减少后续渲染中不必要的片元计算。
    
-   **优化透明物体渲染**：对于透明物体，Depth PrePass 可以在不写入深度的情况下，正确处理遮挡关系。
    

**实现方式**：

1.  **双 Pass 方法**：
    
    -   **Pass 1**：Z-Prepass，只写入深度，不计算颜色。
        
    -   **Pass 2**：关闭深度写入，将深度比较函数设置为相等，进行正常的着色计算。
        
2.  **提前分离的 Prepass**：
    
    -   将 Z-Prepass 分离为单独的 Shader，先渲染场景的深度信息。
        
    -   原来的 Shader 只包含正常的着色计算，关闭深度写入，深度比较函数设置为相等。
        

**优点**：

-   **减少 OverDraw**：显著减少不必要的片元计算，提高性能。
    

**缺点**：

-   **增加 Draw Calls**：由于需要进行两次绘制，Draw Call 和顶点数量翻倍，可能导致性能瓶颈。
    
-   **无法动态批处理**：拥有多个 Pass 的 Shader 无法进行动态批处理，会产生额外的 Draw Call 开销。
    

#### 3. **HiZ（Hierarchical Z，分层深度测试）**

**概念**： HiZ 是一种软件技术，通过创建深度图的多级 Mipmaps 来减少深度比较的次数。具体来说，HiZ 会根据物体的包围盒所在的屏幕坐标与深度图比较深度，如果被挡住就不提交数据给 GPU 渲染。

**用途**：

-   **减少 OverDraw**：通过分层深度测试，减少不必要的深度比较，提高渲染性能。
    
-   **优化透明物体渲染**：HiZ 可以在不写入深度的情况下，正确处理透明物体的遮挡关系。
    

**实现方式**：

-   创建深度图的多级 Mipmaps，每个 Mipmap 层级的像素值取自上一层级的多个像素，记录最远离相机的深度值。
    
-   在渲染时，先用低精度的 Mipmap 层级进行粗略的深度比较，如果物体被挡住，则直接剔除。
    

**优点**：

-   **减少深度比较次数**：通过分层深度测试，显著减少深度比较的次数，提高性能。
    

**缺点**：

-   **需要额外的内存**：存储多级 Mipmaps 需要额外的内存。
    

#### 4. **Late Z（延迟深度测试）**

**概念**： Late Z 是传统的深度测试方式，即在片元着色器阶段进行深度测试。所有片元都会经过片元着色器计算，然后进行深度测试，通过的片元写入颜色缓冲区。

**用途**：

-   **确保正确的遮挡关系**：通过在片元着色器阶段进行深度测试，确保最终的渲染结果正确。
    

**优点**：

-   **简单直接**：实现简单，不需要额外的优化逻辑。
    

**缺点**：

-   **性能较低**：所有片元都会经过片元着色器计算，即使最终被遮挡，也会造成不必要的计算。
    

### 总结

这些深度测试优化技术（Early Z、Depth PrePass、HiZ 和 Late Z）都旨在减少不必要的计算和内存访问，提高渲染性能。它们可以统称为**深度测试优化技术**或**遮挡剔除技术**。在实际应用中，选择哪种技术取决于具体的场景和性能需求。例如，Early Z 和 Depth PrePass 适用于减少 OverDraw，HiZ 适用于减少深度比较次数，而 Late Z 则确保最终的渲染结果正确。
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTgwNzYzMDU2Niw1ODMwMTQxMDEsNDYxMT
Q5MTYyXX0=
-->