Avec `Vulkan` les fonctions ne sont pas directement exécutées via des appels mais stockées  dans des [[Command Buffer]]. Cela permet une meilleure optimisation de l'exécution, car toutes les commandes sont connues à l'avance, et permet aussi d'enregistrer les commandes depuis plusieurs threads.

La création d'un [[Command Buffer]] se fait de cette manière : 
```cpp
VkCommandBufferAllocateInfo alloc_info{};
alloc_info.sType = VK_STRUCTURE_TYPE_COMMAND_BUFFER_ALLOCATE_INFO;
alloc_info.commandPool = m_command_pool;
alloc_info.level = VK_COMMAND_BUFFER_LEVEL_PRIMARY;
alloc_info.commandBufferCount = 1;

if(vkAllocateCommandBuffers(m_device, &alloc_info, &m_command_buffer) != VK_SUCCESS)
	throw std::runtime_error("failed to allocate command buffers");	
```

Avec `VkCommandBufferAllocateInfo` :
- `VkStructureType sType` : Type de la structure
- `VkCommandPool commandPool` : La [[Command Pool]] de notre [[Command Buffer]]
- `uint32_t commandBufferCount` : Le nombre de [[Command Buffer]] à allouer
- `VkCommandBufferLevel level` : Définit comment peut être appelé le [[Command Buffer]]. 
	- `VK_COMMAND_BUFFER_LEVEL_PRIMARY` : peut être directement soumis à une Queue et peut exécuter des `secondary` command buffers.
	- `VK_COMMAND_BUFFER_LEVEL_SECONDARY` : ne peut pas être soumis directement, mais peut être appelé depuis un `primary`.

Nous devons ensuite record les commands dans le [[Command Buffer]].

On peut démarrer le record de cette manière :
```cpp
VkCommandBufferBeginInfo begin_info{};
begin_info.sType = VK_STRUCTURE_TYPE_COMMAND_BUFFER_BEGIN_INFO;

if(vkBeginCommandBuffer(command_buffer, &begin_info) != VK_SUCCESS) {
	throw std::runtime_error("failed to begin recording command buffer");
}
```

Pour record les commands il faut utiliser les fonctions `vkCmd...`. 

On finira ensuite le record de cette manière :
```cpp
if(vkEndCommandBuffer(command_buffer) != VK_SUCCESS) {
	throw std::runtime_error("failed to end command buffer");
}
```

Par exemple le record d'un [[Command Buffer]] basique : 

```cpp
VkCommandBufferBeginInfo begin_info{};
begin_info.sType = VK_STRUCTURE_TYPE_COMMAND_BUFFER_BEGIN_INFO;

if(vkBeginCommandBuffer(
	command_buffer,
	&begin_info) != VK_SUCCESS) {
	throw std::runtime_error("failed to begin recording command buffer");
}

VkRenderPassBeginInfo render_pass_info{};
render_pass_info.sType = VK_STRUCTURE_TYPE_RENDER_PASS_BEGIN_INFO;
render_pass_info.renderPass = m_render_pass;
render_pass_info.framebuffer = m_swap_chain_framebuffers[image_index];
render_pass_info.renderArea.offset = { 0, 0 };
render_pass_info.renderArea.extent = m_swap_chain_extent;

VkClearValue clear_color = { 0.0f, 0.0f, 0.0f, 1.0f };
render_pass_info.clearValueCount = 1;
render_pass_info.pClearValues = &clear_color;

vkCmdBeginRenderPass(
	command_buffer, &render_pass_info, VK_SUBPASS_CONTENTS_INLINE
	);
vkCmdBindPipeline(
	command_buffer, VK_PIPELINE_BIND_POINT_GRAPHICS, m_graphics_pipeline
	);

vkCmdDraw(command_buffer, 3, 1, 0, 0);
vkCmdEndRenderPass(command_buffer);

if(vkEndCommandBuffer(command_buffer) != VK_SUCCESS) {
	throw std::runtime_error("failed to end command buffer");
}
```