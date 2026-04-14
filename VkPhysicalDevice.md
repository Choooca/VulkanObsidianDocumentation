Notre machine peut posséder plusieurs cartes graphiques (GPU, celle intégrée au CPU etc.). Nous devons donc sélectionner le plus appropriée pour notre programme. Chaque GPU peut supporter des [[Extensions]] et des [[Queue Families]] différentes

Pour récupérer tous les [[VkPhysicalDevice]] disponibles nous pouvons utiliser : 

```cpp
uint32_t physical_device_count = 0;
vkEnumeratePhysicalDevices(m_instance, &physical_device_count, nullptr);

std::vector<VkPhysicalDevice> physical_devices(physical_device_count);
vkEnumeratePhysicalDevices(
	m_instance,
	&physical_device_count,
	physical_devices.data()
	);
```

Avec :
- `VkInstance m_instance` : La [[VkInstance]] de notre application
- `uint32_t *physical_device_count` : Un pointeur vers le nombre de [[VkPhysicalDevice]] sur la machine.
- `VkPhysicalDevice *physical_devices` : Un pointeur vers une Array de [[VkPhysicalDevice]]

Pour récupérer les propriétés et les features disponibles dans un device nous pouvons utiliser les méthodes suivantes : 

```cpp
vkGetPhysicalDeviceProperties(physical_device, &physical_device_properties); vkGetPhysicalDeviceFeatures(physical_device, &physical_device_features);
```

Avec : 
 - `VkPhysicalDevice physical_device` :  Le device physique.
 - `VkPhysicalDeviceProperties *physical_device_properties` : Un pointeur nous servant à récupérer les propriétés  
 - `VkPhysicalDeviceFeatures *physical_device_features` : Un pointeur nous servant à récupérer les features

Et on pourra ensuite l'utiliser par exemple de cette manière : 
```cpp
return 
physical_device_properties.deviceType == VK_PHYSICAL_DEVICE_TYPE_DISCRETE_GPU &&
physical_device_features.geometryShader;
```
Pour vérifier si un [[VkPhysicalDevice]] est approprié pour notre application.

Nous n'avons pas besoin de le détruire, il se détruit implicitement avec la destruction du [[VkInstance]]

Ressources  : 
- [Vulkan Tutorial](https://vulkan-tutorial.com/Drawing_a_triangle/Setup/Physical_devices_and_queue_families)
-  [Spécification Vulkan - 5.1 Physical Device](https://registry.khronos.org/vulkan/specs/latest/html/vkspec.html#devsandqueues-physical-device-enumeration)