Vulkan est une API Agnostique, il ne peut alors pas interagir avec la Window par lui même. Nous devons donc utiliser une [[Extensions]] WSI (Window System Integration), en l'occurrence `VK_KHR_surface` qui nous permet alors d'avoir une [[VkSurfaceKHR]]

Une [[VkSurfaceKHR]] est une abstraction représentant la zone de notre fenêtre ou notre image sera rendue.

Normalement le setup peut être un peu compliqué dû au fait que la surface dépend de l'OS utilisé. Mais GLFW gère ça pour nous.
Nous pouvons alors simplement : 

```cpp 
VkResult glfwCreateWindowSurface(
	VkInstance instance,
	GLFWwindow* handle,
	const VkAllocationCallbacks* allocator,
	VkSurfaceKHR* surface
)
```

Avec : 
- `VkInstance instance` : La [[VkInstance]] de notre application.
- `GLFWwindow* handle` : Pointeur de notre window GLFW
- `const VkAllocationCallbacks* allocator` : Custom Allocator
- `VkSurfaceKHR* surface` : Pointeur vers une surface  (valeur de retour)

Et en sélectionnant notre [[VkPhysicalDevice]] nous devons vérifier si une de nos [[Queue Families]] supporte la présentation sur cette surface :
```cpp
VkResult vkGetPhysicalDeviceSurfaceSupportKHR(
    VkPhysicalDevice physical_device,
    uint32_t queue_family_index,
    VkSurfaceKHR surface,
    VkBool32* supported
);
```
Avec : 
- `VkPhysicalDevice physical_device` : Le [[VkPhysicalDevice]] testé
- `uint32_t queue_family_index` : La  [[Queue Families]] à tester pour savoir si elle supporte les features de présentation.
- `VkSurfaceKHR surface` : Notre [[VkSurfaceKHR]]
- `VkBool32* supported` : Boolean de retour pour savoir si notre [[VkPhysicalDevice]] et notre [[Queue Families]] supporte la présentation sur cette [[VkSurfaceKHR]].

La [[VkSurfaceKHR]] doit être détruite avant la [[VkInstance]] via :

```cpp
void vkDestroySurfaceKHR( 
	VkInstance instance,
	VkSurfaceKHR surface,
	const VkAllocationCallbacks* pAllocator
)
```

Ressources : 
- [Vulkan Tutorial](https://vulkan-tutorial.com/Drawing_a_triangle/Presentation/Window_surface)