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