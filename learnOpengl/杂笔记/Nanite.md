# 为什么需要 Nanite？

在传统实时渲染管线里（Unity / UE4 / 其他引擎）：

1.  **模型多边形太多会爆炸**
    
    -   CPU 要给每个物体下发 draw call。
        
    -   GPU 要处理海量顶点和三角形。
        
    -   硬件通常只能实时渲染几百万到几千万三角形。
        
2.  **传统优化手段**
    
    -   **LOD（Level of Detail）**：远处用低模，近处用高模。
        
    -   **烘焙贴图**：法线/置换/细节通过贴图模拟，而不是几何体。
        
    -   **静态合批 / Instancing**：减少 draw call。
        
    -   但：LOD 需要美术制作多个版本，烘焙贴图丢失真实几何细节，合批无法解决面数太高的问题。
        
 **结果**：在电影级高精度资源（数亿多边形）下，传统做法完全吃不消。
 
# Nanite 的核心目标

-   允许开发者直接把 **电影级模型**（ZBrush 高模、CAD 模型，数亿三角）拖进 UE5 而不用手动做 LOD 或烘焙。
    
-   引擎自动保证：
    
    -   看起来和高模一模一样。
        
    -   运行时仍然保持高帧率。
        

Epic 宣传时说的口号就是：**“无限几何细节”**。

# Nanite 的核心原理

Nanite 的突破点就是：  
**不再用 CPU 管理 draw call，而是用 GPU 自己管理几何体。**

具体有几大关键点：

----------

### 1. **Cluster / Meshlet 拆分**

-   把原始模型分解成很多小块（Cluster），每个大概 **128 三角形** 左右。
    
-   每个 Cluster 都存储自己的 **边界包围盒、层级信息**。
    
-   所以一个上亿三角形的模型，会被拆成几十万个 Cluster。
    

👉 **目的**：可以像八叉树 / BVH 一样进行层级管理和剔除。

----------

### 2. **层级 LOD（Hierarchical LOD）**

-   Nanite 会在离线阶段生成一个 **Cluster 层级结构**：
    
    -   近距离 → 高分辨率 Cluster
        
    -   远距离 → 合并后的低分辨率 Cluster
        
-   渲染时，根据相机距离 / 屏幕像素大小，GPU 只画必要的 Cluster。
    
-   和传统 LOD 不同：**这是自动生成的，无需手工建模**。
    

----------

### 3. **GPU 驱动的剔除和选择**

-   在传统渲染里，CPU 要做：
    
    -   视锥体剔除（Frustum Culling）
        
    -   LOD 选择
        
    -   提交 draw call
        
-   **Nanite 让 GPU 来做这些**：
    
    -   一个 Compute Shader 先批量判断哪些 Cluster 可见。
        
    -   结果写到一个 **Command Buffer**（绘制命令缓冲区）。
        
    -   然后 GPU 用 **Multi-Draw Indirect (MDI)** 一次性画所有可见的 Cluster。
        

👉 **CPU 几乎不参与 draw call 调度**，完全是 GPU 自己驱动渲染。

----------

### 4. **Micropolygon Rasterizer（微多边形光栅化）**

-   Nanite 内部并不是用传统的三角形光栅化，而是对 Cluster 里的三角形做了 **屏幕空间自适应细分**。
    
-   大量小三角形会被处理成“微多边形”流 → 只在像素级别需要的地方计算。
    
-   这样保证：
    
    -   不会浪费在超小三角形的 overdraw 上。
        
    -   远处的几何体也能保持清晰而不卡顿。
        

----------

# Nanite 渲染管线（简化版）

1.  **离线预处理**：把模型切成 Cluster，构建层级 LOD。
    
2.  **运行时**：
    
    -   Compute Shader → 剔除不可见 Cluster。
        
    -   Compute Shader → 为可见 Cluster 选择合适的 LOD。
        
    -   写入 **Indirect Command Buffer**。
        
3.  **GPU 一次 Multi-Draw Indirect** → 批量渲染所有 Cluster。
    
4.  光栅化时，Nanite 自适应生成“微多边形”，保证像素级精度。
    

----------

# Nanite 的优势

✅ **电影级资产直接用**：美术不再需要烘焙法线贴图、做 LOD。  
✅ **超高面数可实时渲染**：可以在场景里放几亿多边形，依旧流畅。  
✅ **GPU 驱动管线**：CPU 不再成为瓶颈。  
✅ **自动 LOD**：不用人工干预，随距离自适应。  
✅ **减少内存带宽压力**：Cluster + 压缩存储，比直接加载全精度网格要轻。

----------

# Nanite 的限制

⚠️ **只支持静态网格**（Static Mesh），不能用于 Skeletal Mesh（骨骼动画）。  
⚠️ **不支持变形动画**（例如布料、软体、Morph Target）。  
⚠️ **几何体必须先经过 Nanite 预处理**（不是所有导入的模型都能直接跑）。  
⚠️ **显存占用较大**（因为要存 Cluster 层级数据 + 压缩几何）。

----------

# 总结一句

> **Nanite = GPU 驱动的层级化微多边形渲染技术**
> 
> -   把几何体切分成 Cluster
>     
> -   GPU 做剔除 + LOD
>     
> -   用 Multi-Draw Indirect 一次性渲染
>     
> -   保证像素级几何细节
>     

它解决了“**实时渲染海量三角形**”的老问题，让开发者可以直接用电影资产放进游戏里。
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE3MzQ5NjU0MzBdfQ==
-->