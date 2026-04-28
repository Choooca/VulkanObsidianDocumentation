
Avant de créer nos [[Command Buffer]] nous devons créer une [[Command Pool]] qui détermine la [[Queue Families]] à laquelle les [[Command Buffer]] seront soumis, ainsi que la stratégie d'allocation mémoire

```cpp
QueueFamilyIndices queue_family_indices = FindQueueFamilies(m_physical_device);

VkCommandPoolCreateInfo pool_info{};
pool_info.sType = VK_STRUCTURE_TYPE_COMMAND_POOL_CREATE_INFO;
pool_info.flags = VK_COMMAND_POOL_CREATE_RESET_COMMAND_BUFFER_BIT;
pool_info.queueFamilyIndex = queue_family_indices.graphics_family.value();

if(vkCreateCommandPool(m_device, &pool_info, nullptr, &m_command_pool) !=
	VK_SUCCESS)
	throw std::runtime_error("failed to create command pool");
```

La struct de création `VkCommandPoolCreateInfo`: 
- `VkStructureType sType` : Le type de la struct 
- `uint32_t queueFamilyIndex` : L'index de la [[Queue Families]] définissant les commandes utilisables dans nos [[Command Buffer]]
- `VkCommandPoolCreateFlags flags` : Définit l'utilisation que l'on compte faire de notre pool (la fréquence de la modification de nos [[Command Buffer]] par exemple). Permet de changer le comportement de l'allocation mémoire.

`VkCommandPoolCreateFlags` peut avoir plusieurs valeurs :
- `VK_COMMAND_POOL_CREATE_TRANSIENT_BIT` : Nos [[Command Buffer]] vont être souvent réécrits ou free.
- `VK_COMMAND_POOL_CREATE_RESET_COMMAND_BUFFER_BIT` : Permet de reset nos [[Command Buffer]] individuellement

Et pour la supprimer :

```cpp
vkDestroyCommandPool(m_device, m_command_pool, nullptr);
```
