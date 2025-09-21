# 1. Lumen 的定位

在 **UE4** 时代：

-   静态光照依赖 **Lightmass（离线烘焙光照贴图）**。
    
-   动态光照只能依赖 **距离场 GI (DFGI)**、**SSGI** 或非常贵的 **RTX GI**。
    

在 **UE5**：

-   **Lumen 替代 Lightmass**，让你在不烘焙的情况下得到动态 GI、动态反射。
    
-   目标是“**开箱即用的实时全局光照**”。
    

----------

# 🔹 2. Lumen 的核心思想

Lumen 并不是单一算法，而是一个 **混合方案**，结合了多个技术：

1.  **屏幕空间追踪 (Screen Tracing)**
    
    -   在 GBuffer/Depth Buffer 上做 Ray March，利用现有屏幕像素计算间接光、反射。
        
    -   类似 SSGI / SSR，优点是精细，缺点是屏幕外信息缺失。
        
2.  **场景表示 (Scene Representation)**  
    UE5 为了补全屏幕外数据，构建了一个全局的简化场景表示：
    
    -   **Signed Distance Field (SDF)**：用来快速做光线-场景交集检测。
        
    -   **Mesh Cards (Surface Cache)**：把场景几何投影到多个方向的卡片纹理里，存储间接光照。
        
    -   这种 Surface Cache 就像一个低分辨率的“光照代理”。
        
3.  **层次化追踪 (Hierarchical Tracing)**
    
    -   先在屏幕空间 trace，如果失败就 fallback 到 SDF / Surface Cache。
        
    -   使用层次化的结构（Clipmap / MipMap）做快速光线步进。
        
4.  **多 Bounce 支持**
    
    -   Lumen 不只算一次间接光，还能迭代多次反射/折射。
        
    -   类似传统光线追踪 GI，但通过缓存 + 近似算法保持性能。
        

----------

# 🔹 3. Lumen 提供的能力

1.  **动态全局光照 (Dynamic GI)**
    
    -   光源移动、物体移动，间接光会实时更新。
        
    -   比如拿个手电筒照到白墙，墙的反射光能立刻点亮周围。
        
2.  **动态反射 (Dynamic Reflection)**
    
    -   屏幕空间里是 SSR（高精度）。
        
    -   屏幕外 fallback 到 Lumen 的场景代理，解决 SSR 掉帧和缺失问题。
        
3.  **与 Nanite 集成**
    
    -   Nanite 可以提供高精度几何信息，Lumen 直接利用它的 Proxy Mesh 来更新 Surface Cache。
        
4.  **多平台支持**
    
    -   高端 GPU 可以用 **硬件 RT** 加速 Lumen。
        
    -   低端 GPU/主机仍然可以用 **SDF + Surface Cache** 的软件模式。
        

----------

# 🔹 4. 值得推敲的点

-   **Scene Representation 的设计**
    
    -   SDF 用于遮挡检测，Mesh Card Cache 用于存光照。
        
    -   类似于把场景做了一个“低模光照代理”。
        
-   **混合追踪策略**
    
    -   屏幕空间 → SDF → Surface Cache，逐层 fallback。
        
-   **时序累积 (Temporal Accumulation)**
    
    -   Lumen 的结果需要 TAA 稳定化，否则会闪烁。
        
-   **与 Virtual Shadow Maps/VSM 的关系**
    
    -   Lumen 提供间接光 + 阴影，VSM 提供主光源阴影，它们协作实现完整的光照系统。
        

----------

# 🔹 5. 一句话总结

👉 **Lumen = UE5 的实时全局光照与反射系统**  
它通过 **屏幕空间追踪 + 距离场 + Surface Cache** 混合实现，在保证性能的同时，支持 **动态光源、动态几何、无限场景规模**，完全取代了 UE4 的 Lightmass 烘焙。
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTI3ODc1MTk1XX0=
-->