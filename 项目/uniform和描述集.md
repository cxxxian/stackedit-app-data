在 Vulkan 中，`uniform` 是着色器中用来存储 **常量数据** 的变量，通常用于传递数据给着色器程序，例如：模型矩阵、视图矩阵、投影矩阵等。而 **绑定集（Descriptor Set）** 则是将这些资源（包括 `uniform` 数据）绑定到着色器的机制。简而言之，`uniform` 变量的数据存储在 **缓冲区（Uniform Buffer Object，UBO）** 中，而 UBO 通过 **描述符集** 绑定到 GPU 着色器。

### Uniform 和绑定集的关系

1. **Uniform 变量**：它们定义在 GLSL 着色器代码中，通常用于从 CPU 传递不随顶点或片段变化的数据给着色器。例如，所有顶点都可能使用相同的投影矩阵，这个矩阵就可以通过一个 `uniform` 变量传递给着色器。
   
   在 GLSL 中的 `uniform` 例子：
   ```glsl
   layout(set = 0, binding = 0) uniform UBO {
       mat4 model;
       mat4 view;
       mat4 proj;
   } ubo;
   ```

2. **Uniform Buffer Object (UBO)**：这是在 CPU 端创建的缓冲区，用来存储 `uniform` 变量的数据。它允许在 CPU 和 GPU 之间传递数据，尤其适合存储大小较小的常量数据，比如矩阵、灯光参数等。

3. **Descriptor Set**：描述符集是一个绑定点，用来将 GPU 中的资源（如 `uniform buffer`、纹理等）传递给着色器。通过描述符集，`uniform` 数据（例如从 UBO 传递的数据）可以被绑定到特定的着色器，并被 GPU 在渲染过程中访问。

### Uniform 和绑定集的制作流程

1. **定义 Uniform 变量（在着色器中）**：
   - 在 GLSL 着色器中定义 `uniform` 变量，通常是矩阵、向量等常量数据。
   - 使用 `layout(set = X, binding = Y)` 标注资源的绑定位置，`set` 是描述符集的索引，`binding` 是描述符集内部的资源绑定位置。
   
   例如：
   ```glsl
   layout(set = 0, binding = 0) uniform UBO {
       mat4 model;
       mat4 view;
       mat4 proj;
   } ubo;
   ```

2. **创建 Uniform Buffer Object (UBO)**：
   - 在 CPU 端创建 `VkBuffer` 对象，作为 `uniform buffer` 用来存储数据。这一步是通过 Vulkan API 在主机端分配内存，并将数据写入 `UBO` 中。
   - 常见的 `VkBuffer` 创建代码：
   ```cpp
   VkBufferCreateInfo bufferInfo = {};
   bufferInfo.sType = VK_STRUCTURE_TYPE_BUFFER_CREATE_INFO;
   bufferInfo.size = sizeof(uboData);  // 例如，uboData 是一个包含模型、视图、投影矩阵的结构
   bufferInfo.usage = VK_BUFFER_USAGE_UNIFORM_BUFFER_BIT;
   bufferInfo.sharingMode = VK_SHARING_MODE_EXCLUSIVE;

   VkBuffer uniformBuffer;
   vkCreateBuffer(device, &bufferInfo, nullptr, &uniformBuffer);
   ```

3. **创建 Descriptor Set Layout（描述符集布局）**：
   - 在 Vulkan 中，`Descriptor Set Layout` 定义了每个 `Descriptor Set` 中有哪些资源、这些资源的类型（如 `uniform buffer`、纹理等）以及绑定位置。这里需要定义 `uniform buffer` 的描述符集布局。
   - 示例代码：
   ```cpp
   VkDescriptorSetLayoutBinding uboLayoutBinding = {};
   uboLayoutBinding.binding = 0;  // 对应 GLSL 中的 binding = 0
   uboLayoutBinding.descriptorType = VK_DESCRIPTOR_TYPE_UNIFORM_BUFFER;
   uboLayoutBinding.descriptorCount = 1;
   uboLayoutBinding.stageFlags = VK_SHADER_STAGE_VERTEX_BIT;
   ```

4. **分配 Descriptor Set（描述符集）**：
   - 使用之前创建的 `Descriptor Set Layout` 分配 `Descriptor Set`。描述符集用于绑定具体的资源，比如将 `UBO` 和 `uniform` 变量进行绑定。
   - 示例代码：
   ```cpp
   VkDescriptorSetAllocateInfo allocInfo = {};
   allocInfo.sType = VK_STRUCTURE_TYPE_DESCRIPTOR_SET_ALLOCATE_INFO;
   allocInfo.descriptorPool = descriptorPool;
   allocInfo.pSetLayouts = &descriptorSetLayout;
   allocInfo.descriptorSetCount = 1;

   VkDescriptorSet descriptorSet;
   vkAllocateDescriptorSets(device, &allocInfo, &descriptorSet);
   ```

5. **更新 Descriptor Set（绑定资源）**：
   - 将创建好的 `UBO` 绑定到描述符集中，使其能够在渲染管线中被使用。
   - 示例代码：
   ```cpp
   VkDescriptorBufferInfo bufferInfo = {};
   bufferInfo.buffer = uniformBuffer;
   bufferInfo.offset = 0;
   bufferInfo.range = sizeof(uboData);

   VkWriteDescriptorSet descriptorWrite = {};
   descriptorWrite.sType = VK_STRUCTURE_TYPE_WRITE_DESCRIPTOR_SET;
   descriptorWrite.dstSet = descriptorSet;
   descriptorWrite.dstBinding = 0;  // 对应 GLSL 中的 binding = 0
   descriptorWrite.dstArrayElement = 0;
   descriptorWrite.descriptorType = VK_DESCRIPTOR_TYPE_UNIFORM_BUFFER;
   descriptorWrite.descriptorCount = 1;
   descriptorWrite.pBufferInfo = &bufferInfo;

   vkUpdateDescriptorSets(device, 1, &descriptorWrite, 0, nullptr);
   ```

6. **在渲染时绑定 Descriptor Set**：
   - 渲染时，调用 `vkCmdBindDescriptorSets()` 将 `Descriptor Set` 绑定到渲染管线，供着色器访问。
   - 示例代码：
   ```cpp
   vkCmdBindDescriptorSets(commandBuffer, VK_PIPELINE_BIND_POINT_GRAPHICS, pipelineLayout, 0, 1, &descriptorSet, 0, nullptr);
   ```

### 总结
- `uniform` 是着色器中定义的全局变量，通常用于传递不变的常量数据。
- `Uniform Buffer Object (UBO)` 是存储 `uniform` 变量数据的缓冲区，在 CPU 和 GPU 之间传递数据。
- **描述符集** 是将 `uniform` 数据通过缓冲区绑定到 GPU 着色器的机制。
- 流程主要包括定义 `uniform` 变量、创建 UBO、创建描述符集布局、分配和更新描述符集，最后在渲染时进行绑定。
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTI5OTM3MjI4Ml19
-->