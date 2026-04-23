Un [[Render Pass]] est l'un des composants de la [[Graphics Pipeline]] qui décrit la structure des `Attachment` utilisés dans notre rendu (`Depth`, `Stencil`, `Color` etc.). Il décrit également comment nos attachment sont chargés / stockés.

Nous  devons d'abord créer les descriptions de nos `Attachment`. 

```cpp
VkAttachmentDescription color_attachment{};
color_attachment.format = m_swap_chain_image_format;
color_attachment.samples = VK_SAMPLE_COUNT_1_BIT;

color_attachment.loadOp = VK_ATTACHMENT_LOAD_OP_CLEAR;
color_attachment.storeOp = VK_ATTACHMENT_STORE_OP_STORE;

color_attachment.stencilLoadOp = VK_ATTACHMENT_LOAD_OP_DONT_CARE;
color_attachment.stencilStoreOp = VK_ATTACHMENT_STORE_OP_DONT_CARE;

color_attachment.initialLayout = VK_IMAGE_LAYOUT_UNDEFINED;
color_attachment.finalLayout = VK_IMAGE_LAYOUT_PRESENT_SRC_KHR;
```

Avec : 
- `VkFormat format` : Le format des pixels 
- `VkSampleCountFlagBits samples` : Nombre de samples
- `VkAttachmentLoadOp loadOp` : Opération effectuée sur les données `Color`/`Depth` au début du [[Render Pass]].
- `VkAttachmentStoreOp storeOp` : Opération effectuée sur les données `Color`/`Depth` à la fin du [[Render Pass]].
- `VkAttachmentLoadOp stencilLoadOp` : Equivalent au `loadOp` mais pour le `Stencil`.
- `VkAttachmentStoreOp stencilStoreOp` : Equivalent au `storeOp` mais pour le `Stencil`
- `VkImageLayout initialLayout` : Le `Layout` de la `VkImage` avant le début du [[Render Pass]]. Il peut varier en fonction du format ou de l'utilisation de notre `VkImage` (Présentation, copie mémoire etc.)
- `VkImageLayout finalLayout` : Le `Layout` vers lequel la `VkImage` sera automatiquement transitionée à la fin du [[Render Pass]]

Parmi les layouts courants :
- `VK_IMAGE_LAYOUT_UNDEFINED` : Le contenu précédent de la `VkImage` est ignoré 
- `VK_IMAGE_LAYOUT_COLOR_ATTACHMENT_OPTIMAL` : `VkImage` utilisée en tant que `Color Attachment`. 
- `VK_IMAGE_LAYOUT_PRESENT_SRC_KHR` : `VkImage` utilisée pour la présentation
- `VK_IMAGE_LAYOUT_TRANSFER_DST_OPTIMAL` : `VkImage` utilisée lors d'opérations de copie de mémoire

Un [[Render Pass]] peut contenir plusieurs `Sub Pass`. Chaque `Sub Pass` effectue des opérations de rendu séquentielles, pouvant dépendre des résultats des `Sub Pass` précédentes

Chaque `Sub Pass` doit référencer les `Attachment` qu'il utilise. Pour cela il doit référencer des `VkAttachmentReference`. 

```cpp
VkAttachmentReference color_attachment_ref{};
color_attachment_ref.attachment = 0;
color_attachment_ref.layout = VK_IMAGE_LAYOUT_COLOR_ATTACHMENT_OPTIMAL;
```

Avec : 
- `uint32_t attachment` : Index de l'`Attachment`. Il correspond directement au `layout(location = x)`
- `VkImageLayout layout` : Layout de l'`Attachment` pendant ce `Sub Pass`

Nous pouvons maintenant créer le `Sub Pass` : 

```cpp
VkSubpassDescription subpass{};
subpass.pipelineBindPoint = VK_PIPELINE_BIND_POINT_GRAPHICS;
subpass.colorAttachmentCount = 1;
subpass.pColorAttachments = &color_attachment_ref;
```

On doit préciser le type de `Sub Pass` que c'est (`Graphics` ou `Compute` par exemple).

Et on peut maintenant créer le [[Render Pass]] :

```cpp
VkRenderPassCreateInfo render_pass_create_info{};
render_pass_create_info.sType = VK_STRUCTURE_TYPE_RENDER_PASS_CREATE_INFO;
render_pass_create_info.attachmentCount = 1;
render_pass_create_info.pAttachments = &color_attachment;
render_pass_create_info.subpassCount = 1;
render_pass_create_info.pSubpasses = &subpass;

if (vkCreateRenderPass(
		m_device,
		&render_pass_create_info,
		nullptr,
		&m_render_pass
		) != VK_SUCCESS) {
		throw std::runtime_error("failed to create render pass");
	}
```

La destruction du [[Render Pass]] se fait après la [[Graphics Pipeline]] mais avant le [[VkDevice]].