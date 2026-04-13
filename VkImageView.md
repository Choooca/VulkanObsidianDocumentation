Une [[VkImageView]] est une vue d'une `VkImage` créée lors de la création de notre [[VkSwapchainKHR]].
Elle va décrire comment accéder à la `VkImage` et ce qu'on va lire dedans.

Pour créer une [[VkImageView]] nous devons passer via une structure  `VkImageViewCreateInfo` : 

```cpp
VkImageViewCreateInfo create_info{};
create_info.sType = VK_STRUCTURE_TYPE_IMAGE_VIEW_CREATE_INFO;
create_info.image = m_swap_chain_images[i];
create_info.viewType = VK_IMAGE_VIEW_TYPE_2D;
create_info.format = m_swap_chain_image_format;

create_info.components.r = VK_COMPONENT_SWIZZLE_IDENTITY;
create_info.components.g = VK_COMPONENT_SWIZZLE_IDENTITY;
create_info.components.b = VK_COMPONENT_SWIZZLE_IDENTITY;
create_info.components.a = VK_COMPONENT_SWIZZLE_IDENTITY;

create_info.subresourceRange.aspectMask = VK_IMAGE_ASPECT_COLOR_BIT;
create_info.subresourceRange.baseMipLevel = 0;
create_info.subresourceRange.levelCount = 1;
create_info.subresourceRange.baseArrayLayer = 0;
create_info.subresourceRange.layerCount = 1;
```

Avec : 
- `VkStructureType sType` : Type de la structure
- `VkImage image` : Image que l'on veut accéder
- `VkImageViewType viewType` : Type de vue de notre image (1D, 2D, 3D, ou cubemap)
- `VkFormat format` : Format de la `VkImage` déterminé lors de la création de [[VkSwapchainKHR]]
- `VkComponentMapping components` : Permet de switch un channel vers un autre (assigner le bleu au vert par exemple). `VK_COMPONENT_SWIZZLE_IDENTITY` pour ne rien changer
- `VkImageSubresourceRange subresourceRange` :  Décrit l'utilisation de l'image et quelle partie de l'image on va accéder. Par exemple dans le cas d'exemple, elle sera utilisée en tant que color target avec un seul niveau de mipmap et un seul layer.

On peut maintenant créer notre [[VkImageView]] : 
```cpp
vkCreateImageView(device, &create_info, nullptr, &swap_chain_image_views[i])
```
Avec : 
- `VkDevice device` : Notre [[VkDevice]]
- `const VkImageViewCreateInfo* pCreateInfo` : La structure créée plus haut
- `const VkAllocationCallbacks* pAllocator` : Custom allocator
- `VkImageView* pImage` : Pointeur vers une [[VkImageView]]. Valeur de retour

On peut ensuite la détruire à la fin du programme comme ceci : 
```cpp
vkDestroyImageView(device, imageView, nullptr);
```

La destruction d'une [[VkImageView]] doit se faire avant la destruction de la [[VkSwapchainKHR]].