Vulkan possède de multiples extensions  que nous pouvons activer selon nos besoins. Par exemple, pour interagir avec le WSI (Window System Interface) de notre OS, nous avons besoin d'[[Extensions]] spécifiques. Dans ce cas précis on pourra récupérer les [[Extensions]] grâce à une fonction `GLFW` `glfwGetRequiredInstanceExtensions(uint32_t *glfw_extension_count)`. On peut ensuite pour activer ces [[Extensions]] les passer au create_info du [[VkInstance]].

Cependant il est possible que certaines [[Extensions]] ne soient pas disponibles sur une machine, nous pouvons alors vérifier en comparant la liste des [[Extensions]] voulues par l'utilisateur et la liste d'[[Extensions]] disponibles sur le système. 
Pour récupérer la liste des [[Extensions]] disponibles : 

```cpp
uint32_t extension_count = 0; 
vkEnumerateInstanceExtensionProperties(
	nullptr,
	&extension_count,
	nullptr
);

std::vector<VkExtensionProperties> extensions(extension_count);
vkEnumerateInstanceExtensionProperties(
	nullptr,
	&extension_count,
	extensions.data()
	);
```

Avec :
- `pLayerName` : Nom d'un layer pour filtrer les [[Extensions]] récupérées (nullptr pour toutes les [[Extensions]])
- `pPropertyCount*` : Pointeur vers le nombre d'[[Extensions]]
- `pProperties*` : Pointeur vers un tableau de `VkExtensionProperties`. Peut être `nullptr` pour récupérer juste le count

On peut maintenant comparer la liste des [[Extensions]] demandées par l'utilisateur et celles disponibles.

Ressources :
- [Vulkan Tutorial](https://vulkan-tutorial.com/Drawing_a_triangle/Setup/Instance)