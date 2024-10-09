这段代码实现了一个基于Vulkan的渲染器，使用Vulkan中的**Mesh Shading**扩展来优化绘制几何体的流程。代码包含对两种Mesh Shading扩展的支持：`VK_NV_MESH_SHADER`和`VK_EXT_MESH_SHADER`，并通过`vkCmdExecuteCommands`和`vkCmdDrawMeshTasksNV/EXT`进行渲染。

### 代码结构概述
1. **RendererMeshVK 类**：这是一个基于Vulkan的渲染器类，包含对`VK_NV_MESH_SHADER`和`VK_EXT_MESH_SHADER`的支持，定义了渲染流程及命令缓冲区生成逻辑。
2. **TypeNV 和 TypeEXT 类**：这是两个嵌套类，分别表示`VK_NV_MESH_SHADER`和`VK_EXT_MESH_SHADER`两种扩展的类型。每个类都提供了检查扩展可用性、创建渲染器实例、获取优先级和资源的方法。
3. **GenerateCmdBuffers() 方法**：生成Vulkan命令缓冲区，配置绘制流水线，绑定资源并发出绘制命令。
4. **draw() 方法**：每一帧的渲染入口，负责执行前面生成的命令缓冲区。

### 主要函数说明

#### `vkCmdExecuteCommands`
```cpp
vkCmdExecuteCommands(primary, global.meshletBoxes ? 2 : 1, m_cmdBuffers);
```
`vkCmdExecuteCommands`是Vulkan中用于**二级命令缓冲区执行**的函数。它允许将一个或多个已经记录好的二级命令缓冲区插入到当前正在记录的一级命令缓冲区中。通过这种方式，渲染器可以将复杂的渲染操作分成多个命令缓冲区，分别记录不同的渲染操作（例如场景渲染和包围盒绘制），然后在需要时组合执行。

在这段代码中：
- `primary`是主命令缓冲区，正在记录的一级命令缓冲区。
- `m_cmdBuffers`是一个包含两个二级命令缓冲区的数组。
- `global.meshletBoxes`决定是否执行两组命令缓冲区（即是否绘制meshlet的包围盒），如果为`true`，执行两个命令缓冲区，否则只执行第一个。

#### `vkCmdDrawMeshTasksNV` 和 `vkCmdDrawMeshTasksEXT`
```cpp
if (m_isNV) {
    vkCmdDrawMeshTasksNV(cmd, count, 0);
} else {
    vkCmdDrawMeshTasksEXT(cmd, count, 1, 1);
}
```
`vkCmdDrawMeshTasksNV`和`vkCmdDrawMeshTasksEXT`分别是`VK_NV_MESH_SHADER`和`VK_EXT_MESH_SHADER`扩展中的关键函数，用于执行Mesh Shading任务。

- **Mesh Shading**：Vulkan的Mesh Shading模型将传统的顶点和几何着色阶段替换为Mesh Shader和Task Shader，可以对几何体进行更灵活、高效的处理。Mesh Shader直接操作小块几何体（称为meshlets），而Task Shader可以进一步控制Mesh Shader的执行。
  
- **vkCmdDrawMeshTasksNV**：这是`VK_NV_MESH_SHADER`扩展中的绘制命令，用于启动`count`个Mesh Shader任务。每个任务处理一组Meshlet（几何体小块）。
  
- **vkCmdDrawMeshTasksEXT**：这是`VK_EXT_MESH_SHADER`扩展中的绘制命令，参数比NV版多了两个，分别是Task Shader的工作组数量。

在这段代码中，渲染器根据所支持的扩展选择使用`vkCmdDrawMeshTasksNV`或`vkCmdDrawMeshTasksEXT`来执行Mesh Shading任务。

### 总结
- `vkCmdExecuteCommands`用于在一级命令缓冲区中执行二级命令缓冲区，提升命令的重用性和组织复杂性。
- `vkCmdDrawMeshTasksNV/EXT`用于执行Mesh Shader任务，Mesh Shader是一种比传统顶点和几何着色器更灵活的渲染方式，能够处理复杂的几何体。

这些函数一起构建了这个渲染器的核心渲染流程，通过Mesh Shader渲染大量的几何数据。
<!--stackedit_data:
eyJoaXN0b3J5IjpbNTQzNjMzNjc1XX0=
-->