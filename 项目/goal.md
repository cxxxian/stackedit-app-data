# goal

 1. 将三角形变成立方体
 2. 主要修改createCommandBuffers()
 3. 封装函数
    size_t bufferSize = sizeof(vertices[0]) * vertices.size();
    VkPhysicalDeviceProperties properties;
    vkGetPhysicalDeviceProperties(physicalDevice, &properties);
    size_t nonCoherentAtomSize = properties.limits.nonCoherentAtomSize;
    bufferSize = (bufferSize + nonCoherentAtomSize - 1) / nonCoherentAtomSize * nonCoherentAtomSize;

    VkBuffer stagingBuffer;
    VkDeviceMemory stagingBufferMemory;

    // 创建临时缓冲区
    createBuffer(bufferSize, VK_BUFFER_USAGE_TRANSFER_SRC_BIT,
        VK_MEMORY_PROPERTY_HOST_VISIBLE_BIT | VK_MEMORY_PROPERTY_HOST_COHERENT_BIT,
        &stagingBuffer, &stagingBufferMemory);

    // 将顶点数据复制到临时缓冲区
    void* data;
    if (vkMapMemory(logicalDevice, stagingBufferMemory, 0, bufferSize, 0, &data) != VK_SUCCESS) {
        throw std::runtime_error("Failed to map memory for staging buffer!");
    }
    memcpy(data, vertices.data(), sizeof(vertices[0]) * vertices.size()); // 复制数据
    vkUnmapMemory(logicalDevice, stagingBufferMemory); // 解除映射


## temp
把createVertex和createIndex方法做出形参，可以传入vertexBuffer和vertexBuffer2以及index

<!--stackedit_data:
eyJoaXN0b3J5IjpbMTgzMzYyNDY2OSwxNjQ3NzcxODA1LC03NT
k4OTMwNl19
-->