La [[VkInstance]] est le lien entre la librairie et notre application. Elle sert a passer les informations de l'application (via [[VkApplicationInfo]]), mais également à spécifier quelles [[Validation Layers]] ou [[Extensions]] nous souhaitons activer.

Elle se crée via : 
```cpp
vkCreateInstance(
			const VkInstanceCreateInfo* pCreateInfo,
			const VkAllocationCallbacks* pAllocator,
			VkInstance* pInstance
			);
```

Avec :
- `const VkInstanceCreateInfo* pCreateInfo` contenant : 
	- `VkStructureType sType` : type de la structure
	- `const void *pNext` : pointeur vers une structure d'[[Extensions]] (chainage d'extension)
	- `const VkApplicationInfo *pApplicationInfo` : pointeur vers une structure [[VkApplicationInfo]]
	- `uint32_t enabledLayerCount` : nombre de [[Validation Layers]] à activer
	- `const char* const *ppEnabledLayerNames` : pointeur vers un tableau des noms des [[Validation Layers]] à activer
	- `uint32_t enabledExtensionCount` : nombre d'[[Extensions]] à activer
	- `const char* const *ppEnabledExtensionNames` : pointeur vers un tableau des noms des [[Extensions]] à activer
- `const VkAllocationCallbacks *pAllocator` : Custom allocator
- `VkInstance *pInstance` : l'handle de la [[VkInstance]]

Et détruit grâce à : 

```cpp
void vkDestroyInstance(
	VkInstance instance,
	const VkAllocationCallbacks* pAllocator
);
```

La [[VkInstance]] doit être créée en première et détruite en dernière, car les autres objets dépendent d'elle.

Ressources :
- [Vulkan Tutorial](https://vulkan-tutorial.com/Drawing_a_triangle/Setup/Instance)
- [[vkspec.pdf#page=149|Spécification Vulkan - 4.2 Instance]]