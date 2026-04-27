Un [[Framebuffer]] contient les différentes [[VkImageView]] représentant les `Attachment` (`Color`, `Depth`, `Stencil` etc.) explicités lors de la création de notre [[Render Pass]]. Le [[Framebuffer]] que l'on va utiliser lors d'une boucle de rendu va dépendre de la `VkImage` que la [[VkSwapchainKHR]] nous renvoie lors de l'acquisition. On doit alors créer un [[Framebuffer]] par `VkImage` de la [[VkSwapchainKHR]] :

```cpp
for (int i = 0; i < m_swap_chain_image_views.size(); ++i) {
	VkImageView attachments[] = {
		m_swap_chain_image_views[i]
	};
	
	VkFramebufferCreateInfo framebuffer_create_info{};
	framebuffer_create_info.sType = VK_STRUCTURE_TYPE_FRAMEBUFFER_CREATE_INFO;
	framebuffer_create_info.renderPass = m_render_pass;
	framebuffer_create_info.attachmentCount = 1;
	framebuffer_create_info.pAttachments = attachments;
	framebuffer_create_info.width = m_swap_chain_extent.width;
	framebuffer_create_info.height = m_swap_chain_extent.height;
	framebuffer_create_info.layers = 1;

	if (vkCreateFramebuffer(
			m_device,
			&framebuffer_create_info,
			nullptr,
			&m_swap_chain_framebuffers[i]
		) != VK_SUCCESS) {
		throw std::runtime_error("failed to create framebuffer");
	}
}
```

Avec pour `VkFramebufferCreateInfo` :
- `VkStructureType sType` : Le type de structure 
- `VkRenderPass renderPass` : Le [[Render Pass]] auquel est assigné notre [[Framebuffer]]
- `uint32_t attachmentCount` : Nombre d'`Attachment` 
- `const VkImageView* pAttachments` : Pointeur vers la [[VkImageView]] représentant l'`Attachment` 
- `uint32_t width` : Largeur du [[Framebuffer]].
- `uint32_t height`  : Hauteur du [[Framebuffer]]
- `uint32_t layers` : Nombre de layer du [[Framebuffer]], notamment utilisé pour la VR

Et on peut ensuite créer nos [[Framebuffer]] avec :

```cpp
VkResult vkCreateFramebuffer(
    VkDevice device,
    const VkFramebufferCreateInfo* pCreateInfo,
    const VkAllocationCallbacks* pAllocator,
    VkFramebuffer* pFramebuffer
);
```

Avec : 
- `VkDevice device` : Le [[VkDevice]]
- `const VkFramebufferCreateInfo*` : Structure créée plus haut
- `const VkAllocationCallbacks* pAllocator` : Custom Allocator
- `VkFramebuffer* pFramebuffer` : Pointeur vers le [[Framebuffer]], valeur de retour.

On doit les détruire avant le [[Render Pass]] et les [[VkImageView]] : 

```cpp
void vkDestroyFramebuffer(
    VkDevice                                    device,
    VkFramebuffer                               framebuffer,
    const VkAllocationCallbacks*                pAllocator
);
```

