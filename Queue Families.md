Dans Vulkan toutes les opérations (upload de textures, dessiner un triangle etc.) nécessite des  commandes qui seront envoyées dans une queue. Il y a différents types de queue qui sont contenus dans différentes [[Queue Families]] et qui autorisent seulement certaines commandes. Par exemple certaines queue permettent seulement des commandes graphiques et d'autres seulement des transferts de mémoire.

Pour sélectionner le [[VkPhysicalDevice]] nous devons vérifier si les [[Queue Families]] nécessaires à notre application sont disponibles.

Pour récupérer les [[Queue Families]] disponibles  sur nos [[VkPhysicalDevice]] nous pouvons utiliser : 
```cpp
uint32_t queue_family_count = 0;
vkGetPhysicalDeviceQueueFamilyProperties(
	physical_device,
	&queue_family_count,
	nullptr
);

std::vector<VkQueueFamilyProperties> queue_families(queue_family_count);
vkGetPhysicalDeviceQueueFamilyProperties(
	physical_device,
	&queue_family_count,
	queue_families.data()
);
```

Nous avons alors dans nos `VkQueueFamilyProperties`: 
- `VkQueueFlags queueFlags` : BitMask indiquant les types de commandes supportées par la [[Queue Families]].
- `uint32_t queueCount` : Nombre de queues

Après les avoir récupérées on peut vérifier si on a toutes les queues nécessaires à notre application, nous pouvons le faire de cette manière. 

```cpp
struct QueueFamilyIndices {
	std::optional<uint32_t> graphics_family;

	bool IsComplete() {
		return graphics_family.has_value();
	}
};

QueueFamilyIndices indices;
int i = 0;
for (const auto& queue_family : queue_families) {
	if (queue_family.queueFlags & VK_QUEUE_GRAPHICS_BIT) {
		indices.graphics_family = i;
	}

	if(indices.IsComplete())
		break;

	i++;
}
```

On stocke ici l'indice de [[Queue Families]] car ensuite le [[VkDevice]] en aura besoin.

Ressources : 
-  [Vulkan Tutorial](https://vulkan-tutorial.com/Drawing_a_triangle/Setup/Physical_devices_and_queue_families)
-  [Spécification Vulkan - 5.3 Queue](https://registry.khronos.org/vulkan/specs/latest/html/vkspec.html#devsandqueues-queues)