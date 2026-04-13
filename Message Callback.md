Les [[Message Callback]] permettent de gérer manuellement les output des [[Validation Layers]]. Grâce à cela, nous pouvons par exemple trier le type de message qu'on affiche, ou comment on les affiche.

Pour setup le callback nous devons activer l'[[Extensions]] `VK_EXT_debug_utils`

La fonction de callback doit avoir cette signature : 

```cpp
static VKAPI_ATTR VkBool32 VKAPI_CALL debugCallback(
	VkDebugUtilsMessageSeverityFlagBitsEXT messageSeverity,
	VkDebugUtilsMessageTypeFlagsEXT messageType, 
	const VkDebugUtilsMessengerCallbackDataEXT* pCallbackData,
	void* pUserData
	) {}
```

Avec : 
-  `VkDebugUtilsMessageSeverityFlagBitsEXT messageSeverity` : sévérité du message
- `VkDebugUtilsMessageTypeFlagsEXT messageType` : type du message (performance, comportement hors spécification etc.)
-  `VkDebugUtilsMessengerCallbackDataEXT *pCallbackData` : données du callback (message, objet Vulkan relatif à l'erreur, nombre d'objets) 
-  `void *pUserData` : un pointeur permettant à l'utilisateur de passer des données

Pour lier la fonction au callback nous devons créer un objet de type `VkDebugUtilsMessengerEXT`. Afin de créer cet objet nous devons appeler la fonction `vkCreateDebugUtilsMessengerEXT(...)`, cependant, étant une fonction d'extension, la fonction n'est pas chargée automatiquement, nous devons alors créer un proxy de cette manière :

```cpp
VkResult CreateDebugUtilsMessengerEXT(
	VkInstance instance,
	const VkDebugUtilsMessengerCreateInfoEXT* pCreateInfo,
	const VkAllocationCallbacks* pAllocator,
	VkDebugUtilsMessengerEXT* pDebugMessenger) { 
	
	auto func = 
		(PFN_vkCreateDebugUtilsMessengerEXT)vkGetInstanceProcAddr(
		instance,
		"vkCreateDebugUtilsMessengerEXT"
	);
	
	if (func != nullptr) {
		return func(instance, pCreateInfo, pAllocator, pDebugMessenger);
	} else {
		return VK_ERROR_EXTENSION_NOT_PRESENT;
	}
}
```

Grâce à ça nous pouvons appeler : 
```cpp
CreateDebugUtilsMessengerEXT(
	VkInstance instance,
	const VkDebugUtilsMessengerCreateInfoEXT* pCreateInfo,
	const VkAllocationCallbacks* pAllocator,
	VkDebugUtilsMessengerEXT* pDebugMessenger
	)
```

Avec : 
- `VkInstance instance` : [[VkInstance]] de notre application
- `const VkDebugUtilsMessengerCreateInfoEXT *pCreateInfo` : structure de création avec :
	- `VkStructureType sType` : le type de la structure
	- `VkDebugUtilsMessageSeverityFlagsEXT messageSeverity` : le sévérité que nous allons écouter
	- `VkDebugUtilsMessageTypeFlagsEXT messageType` : le type de message que nous allons écouter
	- `PFN_vkDebugUtilsMessengerCallbackEXT pfnUserCallback` : notre fonction callback
- `const VkAllocationCallbacks* pAllocator` : custom allocator 
-  `VkDebugUtilsMessengerEXT* pDebugMessenger`  : notre objet `VkDebugUtilsMessengerEXT` qui stocke l'handle de debug

Pour la destruction de l'handle nous devons faire la même chose pour accéder à la fonction, étant une fonction d'extension : 

```cpp
void DestroyDebugUtilsMessengerEXT(
	VkInstance instance,
	VkDebugUtilsMessengerEXT debugMessenger,
	const VkAllocationCallbacks* pAllocator
	) {
		auto func = (PFN_vkDestroyDebugUtilsMessengerEXT)vkGetInstanceProcAddr(
			instance,
			"vkDestroyDebugUtilsMessengerEXT"
		);
		
		if (func != nullptr) { 
		func(instance, debugMessenger, pAllocator); 
		}
	}
```

Avec les mêmes paramètres déjà expliqués pour la création.

Ca marche déjà mais nous pouvons passer le debug_create_info au `.pNext` à la création de la [[VkInstance]] pour détecter les erreurs lors de la création et destruction de l'instance.

La destruction de l'handle doit se faire avant la destruction du [[VkInstance]].

Ressources : 
- [Vulkan Tutorial](https://vulkan-tutorial.com/Drawing_a_triangle/Setup/Validation_layers)