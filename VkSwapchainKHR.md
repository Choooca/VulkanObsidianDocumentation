Une [[VkSwapchainKHR]] est une queue de `VkImage` qui attendent d'être présentées à l'écran. Une `VkImage` va être présentée à l'écran pendant que d'autres seront en cours de rendu.

Tous les [[VkPhysicalDevice]] ne supportent pas les [[VkSwapchainKHR]], nous devons alors vérifier si notre [[VkPhysicalDevice]] supporte ou non l'[[Extensions]] permettant d'utiliser les [[VkSwapchainKHR]].

Nous pouvons alors dans un premier temps récupérer les extensions supportées par notre [[VkPhysicalDevice]] :
```cpp
uint32_t extension_count;
vkEnumerateDeviceExtensionProperties(
	physical_device,
	nullptr,
	&extension_count,
	nullptr);

std::vector<VkExtensionProperties> available_extensions(extension_count);
vkEnumerateDeviceExtensionProperties(
	physical_device,
	nullptr,
	&extension_count,
	available_extensions.data()
);
```

Ensuite vérifier si notre liste contient l'[[Extensions]] nécessaire : 
```cpp
for(const VkExtensionProperties &available_extension : available_extensions){
	if(std::string(available_extension.extensionName) ==
	   VK_KHR_SWAPCHAIN_EXTENSION_NAME)
		return true;	
}
```
`VK_KHR_SWAPCHAIN_EXTENSION_NAME` est une macro fournie par Vulkan définissant un string représentant l'[[Extensions]].

Une [[VkSwapchainKHR]] nécessite également d'être compatible avec notre [[VkSurfaceKHR]].

Nous devons d'abord récupérer 3 types de paramètres : 
- `VkSurfaceCapabilitiesKHR` : Correspondant au min/max `VkImage` dans notre [[VkSwapchainKHR]] et la taille min/max de notre image
- `VkSurfaceFormatKHR` : Correspondant au format des pixels et au color space
- `VkPresentModeKHR` : Les différentes méthodes de présentation de la [[VkSwapchainKHR]].

Pour récupérer les `VkSurfaceCapabilitiesKHR` :
```cpp
vkGetPhysicalDeviceSurfaceCapabilitiesKHR(
	physical_device,
	surface,
	&capabilities
);
```

Pour récupérer les `VkSurfaceFormatKHR` :
```cpp
uint32_t format_count;
vkGetPhysicalDeviceSurfaceFormatsKHR(
	physical_device,
	surface,
	&format_count,
	nullptr
);

std::vector<VkSurfaceFormatKHR> formats;
if (format_count != 0) {
	formats.resize(format_count);
	vkGetPhysicalDeviceSurfaceFormatsKHR(
		physical_device,
		surface,
		&format_count,
		formats.data()
	);
}
```

Pour récupérer les `VkPresentModeKHR` :
```cpp
uint32_t present_mode_count; 
vkGetPhysicalDeviceSurfacePresentModesKHR(
	physical_device,
	surface,
	&present_mode_count,
	nullptr
);

std::vector<VkPresentModeKHR> present_modes;
if (present_mode_count != 0) {
	present_modes.resize(present_mode_count);
	vkGetPhysicalDeviceSurfacePresentModesKHR(
		physical_device,
		surface,
		&present_mode_count,
		present_modes.data()
	);
}
```

Il faut bien vérifier si minimum un `VkPresentModeKHR` et `VkSurfaceFormatKHR` sont disponibles :
```cpp
!present_modes.empty() && !formats.empty()
```

Après avoir vérifié si notre [[VkPhysicalDevice]] peut supporter une [[VkSwapchainKHR]] nous pouvons sélectionner les bons paramètres pour notre [[VkSwapchainKHR]].

Par exemple si on veut sélectionner un certain `VkSurfaceFormatKHR` : 
```cpp
for (const auto& available_format : available_formats) {
	if(available_format.format == VK_FORMAT_B8G8R8A8_SRGB && 
	   available_format.colorSpace == VK_COLOR_SPACE_SRGB_NONLINEAR_KHR)
			return available_format;
	}
	return available_formats[0];
```
Avec : 
- `VkFormat format` : spécifie les channels et le type des channels (ex: `VK_FORMAT_B8G8R8A8_SRGB` RGBA et 8 bits / channel)
- `VkColorSpaceKHR colorSpace` : si le `colorSpace` est en SRGB ou non

On peut sélectionner le `VkPresentModeKHR` : 
```cpp
for (const auto& available_present_mode : available_present_modes) {
	if (available_present_mode == VK_PRESENT_MODE_MAILBOX_KHR) {
		return available_present_mode;
	}
}

return VK_PRESENT_MODE_FIFO_KHR;
```

Pour connaitre les différents `VkPresentModeKHR` [ici](https://docs.vulkan.org/refpages/latest/refpages/source/VkPresentModeKHR.html)

On doit enfin spécifier le `VkExtent2D` de notre [[VkSwapchainKHR]]. Pour cela il y a deux cas.
Dans la plupart des cas le `VkExtent2D` est égale à la résolution de la fenêtre. On peut alors directement le récupérer de cette manière : 
```cpp
VkSurfaceCapabilitiesKHR.currentExtent
```

Si `currentExtent` vaut `UINT32_MAX`, c'est que la surface laisse le choix à l'application, et l'on doit alors calculer manuellement le `VkExtent2D`. Cela arrive notamment sur les écrans à haut `DPI`, où les pixels physiques ne correspondent plus aux screen coordinates. Nous pouvons alors récupérer la véritable taille de notre [[VkSurfaceKHR]] en utilisant `GLFW` : 

```cpp
int width, height;
glfwGetFramebufferSize(m_window, &width, &height);

VkExtent2D actual_extent = {
	static_cast<uint32_t>(width),
	static_cast<uint32_t>(height)
};

actual_extent.width = std::clamp(actual_extent.width, capabilities.minImageExtent.width, capabilities.maxImageExtent.width);
actual_extent.height = std::clamp(actual_extent.height, capabilities.minImageExtent.height, capabilities.maxImageExtent.height);
```

On clamp ensuite pour  être sûr que c'est bien dans les limites permises par notre implémentation.

Maintenant que nous avons nos spécifications (`VkSurfaceFormatKHR`, `VkPresentModeKHR`, et `VkExtent2D`) nous pouvons créer la [[VkSwapchainKHR]]. 

On doit décider le nombre d'images dans notre [[VkSwapchainKHR]]. Nous pouvons le récupérer le minimum via le `VkSurfaceCapabilitiesKHR` : 

```cpp
uint32_t image_count = swap_chain_support.capabilities.minImageCount + 1;

if (swap_chain_support.capabilities.maxImageCount > 0 &&
	image_count > swap_chain_support.capabilities.maxImageCount) {
	image_count = swap_chain_support.capabilities.maxImageCount;
}
```

Il est conseillé de rajouter une image au minimum afin d'éviter que le driver ne doive parfois attendre de finir certaines opérations avant de pouvoir nous donner une nouvelle image.

Nous créons maintenant notre structure pour la création de notre [[VkSwapchainKHR]] : 

```cpp
VkSwapchainCreateInfoKHR create_info{};
create_info.sType = VK_STRUCTURE_TYPE_SWAPCHAIN_CREATE_INFO_KHR;
create_info.surface = m_surface;
create_info.minImageCount = image_count;
create_info.imageFormat = surface_format.format;
create_info.imageColorSpace = surface_format.colorSpace;
create_info.imageExtent = extent;
create_info.imageArrayLayers = 1;
create_info.imageUsage = VK_IMAGE_USAGE_COLOR_ATTACHMENT_BIT;
create_info.presentMode = present_mode;
create_info.clipped = VK_TRUE;
create_info.compositeAlpha = VK_COMPOSITE_ALPHA_OPAQUE_BIT_KHR;
create_info.preTransform = capabilities.currentTransform;
create_info.oldSwapchain = VK_NULL_HANDLE;
```
Avec :
- `VkStructureType sType` : Le type de la structure
- `VkSurfaceKHR surface` : Notre [[VkSurfaceKHR]]
- `uint32_t minImageCount` : L'image count calculé plus haut 
- `VkFormat imageFormat` : le `.format` du `VkSurfaceFormatKHR` déterminé plus haut
- `VkColorSpaceKHR imageColorSpace` : le `.colorSpace` du `VkSurfaceFormatKHR` déterminé plus haut
- `VkExtent2D imageExtent` : le `VkExtent2D` déterminé plus haut
- `uint32_t imageArrayLayers` : le nombre de layer d'image. Souvent 1 sauf cas spécifique (ex : rendu VR)
- `VkImageUsageFlags imageUsage` : Comment la `VkImage` va être utilisée. Par exemple `VK_IMAGE_USAGE_COLOR_ATTACHMENT_BIT` signifie que l'on va directement rendre sur l'image mais `VK_IMAGE_USAGE_TRANSFER_DST_BIT` signifie que l'on ne rendra pas directement dessus mais que l'on fera un transfert mémoire pour rendre dessus (ex : post processing) 
- `VkPresentModeKHR presentMode` : Le `VkPresentModeKHR` déterminé plus haut.
- `VkBool32 clipped`  : Si les pixels cachés (par une autre fenêtre par exemple) doivent être ignorés (donc pas calculés) ou non. Souvent mis à VK_TRUE pour gagner des performances.
- `VkCompositeAlphaFlagBitsKHR compositeAlpha` : Si notre fenêtre blend avec les autres ou non (souvent laissé à `VK_COMPOSITE_ALPHA_OPAQUE_BIT_KHR`)
- `VkSurfaceTransformFlagBitsKHR preTransform` : Si on doit appliquer une transformation à une image. Si l'on n'en veut aucune, nous pouvons assigner ce paramètre au `.currentTransform` du `VkSurfaceCapabilitiesKHR` déterminé plus haut.
- `VkSwapchainKHR oldSwapchain` : L'ancienne [[VkSwapchainKHR]]. Utile lors de la recréation de [[VkSwapchainKHR]] par exemple lors du resize d'une fenêtre. Elle permet notamment à Vulkan de réutiliser les ressources existantes

On doit ensuite spécifier comment sont gérées les images qui seront utilisées dans plusieurs [[Queue Families]]. Par exemple dessiner une image via la `Graphics Queue` et la présenter à l'écran via la `Presentation Queue`. Ce n'est pas nécessaire dans tous les cas, par exemple si la [[Queue Families]] supportant la `Graphics Queue` supporte également la `Presentation Queue`.

Dans le cas contraire il y a deux modes :
- `VK_SHARING_MODE_EXCLUSIVE` : Une image est détenue par une seule [[Queue Families]] et le transfert de propriété (ownership transfer) doit être fait explicitement
- `VK_SHARING_MODE_CONCURRENT` : Une image peut être utilisée en même temps par plusieurs [[Queue Families]].

En se référant à la structure `QueueFamilyIndices` du chapitre [[Queue Families]] : 

```cpp
QueueFamilyIndices indices = FindQueueFamilies(m_physical_device);
uint32_t queue_family_indices[] = {
	indices.graphics_family.value(),
	indices.present_family.value()
};

if (indices.graphics_family != indices.present_family) {
	create_info.imageSharingMode = VK_SHARING_MODE_CONCURRENT;
	create_info.queueFamilyIndexCount = 2;
	create_info.pQueueFamilyIndices = queue_family_indices;
}
else {
	create_info.imageSharingMode = VK_SHARING_MODE_EXCLUSIVE;
}
```

Avec : 
- `VkSharingMode imageSharingMode` : Mode de partage des images
- `uint32_t queueFamilyIndexCount` : Nombre de [[Queue Families]]
- `const uint32_t* pQueueFamilyIndices` : Pointeur vers un tableau des indices des [[Queue Families]]

On peut désormais créer notre [[VkSwapchainKHR]] : 

```cpp
vkCreateSwapchainKHR(m_device, &create_info, nullptr, &m_swap_chain)
```

Et nous devons la détruire avant notre [[VkDevice]] : 
```cpp
vkDestroySwapchainKHR(m_device, m_swap_chain, nullptr);
```

On peut maintenant stocker nos `VkImage` créées implicitement lors de la création de notre [[VkSwapchainKHR]] : 
```cpp
vkGetSwapchainImagesKHR(
	m_device,
	m_swap_chain,
	&image_count,
	nullptr
);

m_swap_chain_images.resize(image_count);
vkGetSwapchainImagesKHR(
	m_device,
	m_swap_chain,
	&image_count,
	m_swap_chain_images.data()
);
```

Il est également utile de stocker le `VkFormat` et le `VkExtent2D` pour les étapes suivantes.

Les  `VkImage2D` n'ont pas besoin d'être détruite explicitement, elles le sont avec lors de la destruction de la [[VkSwapchainKHR]].