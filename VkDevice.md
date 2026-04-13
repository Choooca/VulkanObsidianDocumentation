Le [[VkDevice]] (ou logical device) est le composant nous permettant d'interfacer le [[VkPhysicalDevice]]. 

Dans un premier temps pour la création du [[VkDevice]] nous devons créer les queues nécessaires en fonction des [[Queue Families]] demandées.

En se référant au code pour récupérer les [[Queue Families]] nous pouvons créer les queues pour le [[VkDevice]] de cette manière : 
```cpp
std::vector<VkDeviceQueueCreateInfo> queue_create_infos;
std::set<uint32_t> unique_queue_families = {
	indices.graphics_family.value(),
	indices.present_family.value()
	};

float queue_priority = 1.0f;
for (uint32_t queue_family : unique_queue_families) {
	VkDeviceQueueCreateInfo queue_create_info{};
	queue_create_info.sType = VK_STRUCTURE_TYPE_DEVICE_QUEUE_CREATE_INFO;
	queue_create_info.queueFamilyIndex = queue_family;
	queue_create_info.queueCount = 1;
	queue_create_info.pQueuePriorities = &queue_priority;
	queue_create_infos.push_back(queue_create_info);
}
```

Avec pour `VkDeviceQueueCreateInfo` :
- `VkStructureType sType` : Le type de la structure
- `uint32_t queueFamilyIndex` : L'index de la [[Queue Families]].
- `uint32_t queueCount` : Le nombre de queue créées pour cette [[Queue Families]]
- `const float *pQueuePriorities` : Valeur allant de 0 à 1 pour prioriser le scheduling des commandes [Queue Priority](https://docs.vulkan.org/spec/latest/chapters/devsandqueues.html#devsandqueues-priority)

On doit ensuite référencer les `VkPhysicalDeviceFeatures` (je ne sais pas encore ce que c'est)
```cpp
VkPhysicalDeviceFeatures deviceFeatures{};
```

Maintenant que l'on a ces deux structures nous pouvons créer le [[VkDevice]]. 

```cpp
VkDeviceCreateInfo create_info{};
create_info.sType = VK_STRUCTURE_TYPE_DEVICE_CREATE_INFO;

create_info.queueCreateInfoCount = 
	static_cast<uint32_t>(queue_create_infos.size());
create_info.pQueueCreateInfos = queue_create_infos.data();

create_info.enabledExtensionCount = 
	static_cast<uint32_t>(m_device_extensions.size());
create_info.ppEnabledExtensionNames = m_device_extensions.data();

create_info.pEnabledFeatures = &device_features;

vkCreateDevice(m_physical_device, &create_info, nullptr, &m_device)
```

Avec : 

- `VkPhysicalDevice physicalDevice` : Notre handle [[VkPhysicalDevice]]
- `const VkDeviceCreateInfo* pCreateInfo` :
	- `VkStructureType sType` : Le type de la structure
	- `uint32_t queueCreateInfoCount` : nombre de `VkDeviceQueueCreateInfo` 
	- `const VkDeviceQueueCreateInfo *pQueueCreateInfos` : Pointeur vers une Array `VkDeviceQueueCreateInfo`.
	- `uint32_t enabledExtensionCount` : Nombre d'extensions à activer
	- `const char* const *ppEnabledExtensionNames` : Pointeur vers une Array correspondant aux [[Extensions]] à activer.
	- `const VkPhysicalDeviceFeatures *pEnabledFeatures` : Pointeur vers le `VkPhysicalDeviceFeatures` créé plus haut.
- `const VkAllocationCallbacks *pAllocator` : Custom allocator 
- `VkDevice *pDevice` : Pointeur vers le handle [[VkDevice]]

Et on pourra le détruire avec : 
```cpp
void vkDestroyDevice(
	VkDevice device,
	const VkAllocationCallbacks* pAllocator
);
```

Maintenant, les queue sont créées, donc afin de stocker les handles de ces queues :

```cpp
void vkGetDeviceQueue( 
	VkDevice device,
	uint32_t queueFamilyIndex,
	uint32_t queueIndex,
	VkQueue* pQueue
);
```

Avec : 
- `VkDevice device` : Notre handle [[VkDevice]]
- `uint32_t queueFamilyIndex` : L'index de notre [[Queue Families]]
- `uint32_t queueIndex` : L'index de notre queue
- `VkQueue* pQueue` : Pointeur vers notre handle Queue

Ressources : 
- [Vulkan Tutorial](https://vulkan-tutorial.com/Drawing_a_triangle/Setup/Logical_device_and_queues)
- [Spécification Vulkan - 5.2 Device](https://registry.khronos.org/vulkan/specs/latest/html/vkspec.html#devsandqueues-devices)