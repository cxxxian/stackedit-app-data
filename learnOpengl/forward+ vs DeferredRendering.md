一、选择 Deferred Rendering 的典型场景
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
4. 性能瓶颈
带宽压力：
```math
复制代码
\text{G-Buffer 带宽} = \text{分辨率} \times \text{每像素字节数} \times \text{Buffer数量}
4K x 4(RGBA16F) x 4 Buffers ≈ 2560x1440x(8 bytes/channel x4 channels)x4 ≈ 567MB/Frame
透明物体处理：
需要额外 Forward Pass 渲染透明物体
5. 典型案例
《赛博朋克 2077》：
密集霓虹灯、车灯、全动态光照
使用改进版 Deferred + Ray Tracing 混合管线
二、选择 Forward+ 的典型场景
1. 核心优势
带宽效率：
不需要存储完整的 G-Buffer
移动端等带宽敏感平台优势明显
材质多样性支持：
复杂材质（毛发、皮肤 SSS）可直接在着色器实现
无 G-Buffer 精度限制（如法线压缩失真）
2. 技术实现关键
cpp
复制代码
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
3. 适用平台
移动端/VR 设备：
Adreno/Mali GPU 的 Tile-Based 架构天然适配
带宽节省示例：
math
复制代码
\text{Forward+ 带宽} ≈ \frac{1}{3} \times \text{Deferred}  
需要 MSAA 的场景：
Forward 管线天然支持硬件 MSAA
Deferred 的 MSAA 需要多重采样 G-Buffer（成本极高）
4. 性能瓶颈
光源数量限制：
每个分块的光源数受限于 GPU 寄存器（通常 ≤ 256）
动态分支过多可能降低 SIMD 利用率
复杂光源类型：
投射阴影的聚光灯处理不如 Deferred 高效
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTcwMjYwNDU5M119
-->