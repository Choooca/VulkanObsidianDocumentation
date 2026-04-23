
La [[Graphics Pipeline]] est la séquence d'opérations transformant les données d'entrée (vertices, indices, textures etc.) en pixels écrits dans un [[Framebuffer]]

![[vulkan_simplified_pipeline.jpg]]

Dans la [[Graphics Pipeline]] il y a deux types d'étapes :

- Les fixed-functions, ce sont des étapes, comme le `Color blending`, qui sont prédéfinies mais qu'on peut configurer à l'aide de paramètres. Elles sont notées en verte sur le schéma.
- Les programmable, ce sont les étapes, comme le `Fragment shader`, où l'utilisateur peut upload son propre code et donc programmer son propre comportement. Elles sont notées en orange sur le schéma.

Si l'on veut changer des paramètres d'une [[Graphics Pipeline]], nous devons la recréer de 0 (sauf dans certains cas vu plus bas). C'est contraignant mais ça permets à Vulkan de faire des optimisations car il connait déjà toutes les étapes de la [[Graphics Pipeline]].
## Dynamic State

Vulkan permet de spécifier des states spécifiques que l'on pourra modifier sans recréer la [[Graphics Pipeline]]. C'est souvent le cas pour la taille du `Viewport` ou les constantes de `Blend`. 

```cpp
const std::vector<VkDynamicState> m_dynamic_states = {
	VK_DYNAMIC_STATE_VIEWPORT,
	VK_DYNAMIC_STATE_SCISSOR
};

VkPipelineDynamicStateCreateInfo dynamic_state_create_info{};

dynamic_state_create_info.sType = 
	VK_STRUCTURE_TYPE_PIPELINE_DYNAMIC_STATE_CREATE_INFO;
	
dynamic_state_create_info.dynamicStateCount = 
	static_cast<uint32_t>(m_dynamic_states.size());
	
dynamic_state_create_info.pDynamicStates = 
	m_dynamic_states.data();
```

Avec : 
- `VkStructureType sType` : Type de la structure
- `uint32_t dynamicStateCount` : Nombre de `Dynamic State`
- `const VkDynamicState *pDynamicStates` : Pointeur vers une Array contenant les différents `Dynamic State`.

Dans notre cas le `Viewport` et le `Scissor` sont des `Dynamic State`, il faudra alors les spécifier lors du recording de notre [[Command Buffer]]

## Vertex Input

Les `Vertex Input` décrit le format des vertex (espacement entre instance, type d'attribut etc.). Je reviendrai compléter ici quand j'aurais fais les vertex. 

```cpp
VkPipelineVertexInputStateCreateInfo vertex_input_info{};
vertex_input_info.sType =
VK_STRUCTURE_TYPE_PIPELINE_VERTEX_INPUT_STATE_CREATE_INFO;
vertex_input_info.vertexBindingDescriptionCount = 0;
vertex_input_info.pVertexBindingDescriptions = nullptr;
vertex_input_info.vertexAttributeDescriptionCount = 0;
vertex_input_info.pVertexAttributeDescriptions = nullptr;
```

## Input Assembly

L'`Input Assembly`sert à spécifier la topologie des primitives et si le primitive restart est activé. L'ordre dans lequel les vertices sont connectées est soit déterminé par l'ordre dans le [[Vertex Buffer]], soit défini par un [[Index Buffer]]. Les géométries que nous pouvons dessiner peuvent être des triangles, des lignes, des lignes chaînées etc. 

```cpp
VkPipelineInputAssemblyStateCreateInfo input_assembly_create_info{};
input_assembly_create_info.sType = 
	VK_STRUCTURE_TYPE_PIPELINE_INPUT_ASSEMBLY_STATE_CREATE_INFO;
input_assembly_create_info.topology = VK_PRIMITIVE_TOPOLOGY_TRIANGLE_LIST;
input_assembly_create_info.primitiveRestartEnable = VK_FALSE;
```

Avec :
- `VkStructureType sType` : Type de la structure
- `VkPrimitiveTopology topology` : Type de géométrie
- `VkBool32 primitiveRestartEnable` : Permet de couper une primitive strip en insérant un index spécial (`0xFFFF` pour uint16, `0xFFFFFFFF` pour uint32) dans l'Index Buffer. Ne s'applique qu'aux topologies `_STRIP`

Pour les différents types de topologies [ici](https://docs.vulkan.org/refpages/latest/refpages/source/VkPrimitiveTopology.html)

## Shader Stages

Après avoir compilé puis créé les [[Shader Modules]] nous devons créer le `Shader Stages` de cette manière : 

```cpp
VkPipelineShaderStageCreateInfo vert_shader_stage_info{};
vert_shader_stage_info.sType = VK_STRUCTURE_TYPE_PIPELINE_SHADER_STAGE_CREATE_INFO;
vert_shader_stage_info.stage = VK_SHADER_STAGE_VERTEX_BIT;
vert_shader_stage_info.module = vert_shader_module;
vert_shader_stage_info.pName = "main";

VkPipelineShaderStageCreateInfo frag_shader_stage_info{};
frag_shader_stage_info.sType = VK_STRUCTURE_TYPE_PIPELINE_SHADER_STAGE_CREATE_INFO;
frag_shader_stage_info.stage = VK_SHADER_STAGE_FRAGMENT_BIT;
frag_shader_stage_info.module = frag_shader_module;
frag_shader_stage_info.pName = "main";
```
Avec : 
- `VkShaderStageFlagBits stage` : Le type de stage (vertex, fragment, geometry etc.)
- `VkShaderModule module` : Le shader module
- `const char *pName` : Le nom de la fonction d'entrée

On peut ensuite créer notre `Array` contenant nos stages.

```cpp
VkPipelineShaderStageCreateInfo shader_stages[] = 
	{ vert_shader_stage_info, frag_shader_stage_info };
```

## Viewport And Scissor

Le `Viewport` représente la zone du [[Framebuffer]] dans laquelle nous allons dessiner. Ce sera souvent entre `(0, 0)` et `(width, height)`.

```cpp
VkViewport viewport{};
viewport.x = 0.0f;
viewport.y = 0.0f;
viewport.width = static_cast<float>(m_swap_chain_extent.width);
viewport.height = static_cast<float>(m_swap_chain_extent.height);
viewport.maxDepth = 1.0f;
viewport.minDepth = 0.0f;
```

Avec :
- `float x` : Position en `x` du `Viewport`
- `float y` : Position en `y` du `Viewport`
- `float width` : Taille du `Viewport` en `x`
- `float height` : Taille du `Viewport` en `y`
- `float minDepth` : `Depth` minimum (inclus entre 0 et 1)
- `float maxDepth` : `Depth` maximum (inclus entre 0 et 1)

Le `Scissor` définit un rectangle de clipping, tout pixel en dehors de cette zone sera ignoré par le rasterizer.

La différence entre le `Viewport` et le `Scissor` : 

![[Pasted image 20260420110553.png]]

```cpp
VkRect2D scissor{};
scissor.offset = {0, 0};
scissor.extent = m_swap_chain_extent;
```

Avec : 
- `VkOffset2D offset` : Décalage du `Scissor` par rapport à l'origine du [[Framebuffer]]
- `VkExtent2D extent` : Taille du `Scissor`

Et pour créer la struct : 
```cpp
VkPipelineViewportStateCreateInfo viewport_state_create_info{};
viewport_state_create_info.sType = 
	VK_STRUCTURE_TYPE_PIPELINE_VIEWPORT_STATE_CREATE_INFO;
viewport_state_create_info.viewportCount = 1;
viewport_state_create_info.pViewports = &viewport;
viewport_state_create_info.scissorCount = 1;
viewport_state_create_info.pScissors = &scissor;
```

Si le `Viewport` et le `Scissor` sont déclarés comme `Dynamic States`, nous pouvons laisser le `pViewport` et le `pScissors` à `nullptr` et spécifier leurs `count` uniquement.

## Rasterizer

Le `Rasterizer` récupère les vertices en sortie du `Vertex Shader` pour les transformer en fragments. Il permet aussi d'effectuer différents tests comme le `Face Culling` ou le `Depth Testing`.

Pour créer la struct :
```cpp
VkPipelineRasterizationStateCreateInfo rasterization_create_info{};
rasterization_create_info.sType = 
	VK_STRUCTURE_TYPE_PIPELINE_RASTERIZATION_STATE_CREATE_INFO;
rasterization_create_info.depthClampEnable = VK_FALSE;
rasterization_create_info.rasterizerDiscardEnable = VK_FALSE;
rasterization_create_info.polygonMode = VK_POLYGON_MODE_FILL;
rasterization_create_info.lineWidth = 1.0f;
rasterization_create_info.cullMode = VK_CULL_MODE_BACK_BIT;
rasterization_create_info.frontFace = VK_FRONT_FACE_CLOCKWISE;
rasterization_create_info.depthBiasEnable = VK_FALSE;
rasterization_create_info.depthBiasConstantFactor = 0.0f;
rasterization_create_info.depthBiasClamp = 0.0f;
rasterization_create_info.depthBiasSlopeFactor = 0.0f;
```

Avec : 
- `VkStructureType sType` : Le type de la structure
- `VkBool32 depthClampEnable` : Si `VK_TRUE`, les fragments en dehors du near/far plane sont clampés au lieu d'être discard. Peut être utile dans des cas comme une `Shadow Map`. Nécessite l'activation d'une feature GPU
- `VkBool32 rasterizerDiscardEnable` : Si le paramètre est mis à `VK_TRUE`, discard tous les pixels. Cela résulte en aucun output dans le [[Framebuffer]].
- `VkPolygonMode polygonMode` : Spécifie comment les fragments sont dessinés, il y a trois options : `VK_POLYGON_MODE_FILL` remplit la  zone avec des pixels, `VK_POLYGON_MODE_LINE` trace des lignes entre les vertices, `VK_POLYGON_MODE_POINT` dessine uniquement des points sur les vertices.
- `float lineWidth` : Largeur des lignes, si supérieure à 1, nécessite l'activation d'une feature GPU.
- `VkCullModeFlags cullMode` : Définit le type de `Face Culling` voulu (front, back, ou désactivé).
- `VkFrontFace frontFace` : Spécifie l'ordre des vertices représentant les front faces (clockwise ou counter-clockwise)
- `VkBool32 depthBiasEnable` : Active le `Depth Bias` ou non, utile pour les `Shadow Map`.
- `float depthBiasConstantFactor` : Paramètre du Bias
- `float depthBiasClamp` : Paramètre du Bias 
- `float depthBiasSlopeFactor` : Paramètre du Bias

## Multisampling

Le multisampling est une manière d'effectuer de l'`Anti Aliasing`. Je détaillerai ça plus tard 

```cpp
VkPipelineMultisampleStateCreateInfo multisampling_create_info{};
multisampling_create_info.sType = 
	VK_STRUCTURE_TYPE_PIPELINE_MULTISAMPLE_STATE_CREATE_INFO;
multisampling_create_info.sampleShadingEnable = VK_FALSE;
multisampling_create_info.rasterizationSamples = VK_SAMPLE_COUNT_1_BIT;
multisampling_create_info.minSampleShading = 1.0f;
multisampling_create_info.pSampleMask = nullptr;
multisampling_create_info.alphaToCoverageEnable = VK_FALSE;
multisampling_create_info.alphaToOneEnable = VK_FALSE;
```
## Depth & Stencil

## Color Blending

Quand un fragment est envoyé par le fragment shader il faut ensuite qu'il se `Blend` avec celui déjà stocké dans le [[Framebuffer]]. On peut soit mixer les couleurs avec cette opération : 

```cpp
finalColor.rgb = 
	(srcColorBlendFactor * newColor.rgb)
	<colorBlendOp>
	(dstColorBlendFactor * oldColor.rgb);

finalColor.a =
	(srcAlphaBlendFactor * newColor.a)
	<alphaBlendOp>
	(dstAlphaBlendFactor * oldColor.a);
```

Soit avec une opération bitwise. 

Dans un premier temps nous devons définir un `VkPipelineColorBlendAttachmentState` par `Color Attachment` vers lequel on va écrire, on aura donc un `Blend` par `Color Attachment`. Par exemple si on a un `Color Attachment` pour les normal, un autre pour les tangent et un autre pour les positions, nous aurons 3 `VkPipelineColorBlendAttachmentState`.

```cpp
VkPipelineColorBlendAttachmentState color_blend_attachment{};
color_blend_attachment.colorWriteMask = 
	VK_COLOR_COMPONENT_R_BIT |
	VK_COLOR_COMPONENT_G_BIT |
	VK_COLOR_COMPONENT_B_BIT |
	VK_COLOR_COMPONENT_A_BIT;
color_blend_attachment.blendEnable = VK_TRUE;
color_blend_attachment.srcColorBlendFactor = VK_BLEND_FACTOR_SRC_ALPHA;
color_blend_attachment.dstColorBlendFactor = VK_BLEND_FACTOR_ONE_MINUS_SRC_ALPHA;
color_blend_attachment.colorBlendOp = VK_BLEND_OP_ADD;
color_blend_attachment.srcAlphaBlendFactor = VK_BLEND_FACTOR_ONE;
color_blend_attachment.dstAlphaBlendFactor = VK_BLEND_FACTOR_ZERO;
color_blend_attachment.alphaBlendOp = VK_BLEND_OP_ADD;
```

Avec : 
- `VkBool32 blendEnable` : Contrôle l'activation du `Blend`
- `VkColorComponentFlags colorWriteMask` : Quels channels sont écrits dans l'`Attachment`

La struct ci-dessus représente un `alpha blending`.

Ensuite nous devons créer un `VkPipelineColorBlendStateCreateInfo` contenant tous nos `VkPipelineColorBlendAttachmentState` : 

```cpp
VkPipelineColorBlendStateCreateInfo color_blending_create_info{};
color_blending_create_info.sType =
	VK_STRUCTURE_TYPE_PIPELINE_COLOR_BLEND_STATE_CREATE_INFO;
color_blending_create_info.logicOpEnable = VK_FALSE;
color_blending_create_info.logicOp = VK_LOGIC_OP_COPY;
color_blending_create_info.attachmentCount = 1;
color_blending_create_info.pAttachments = &color_blend_attachment;
color_blending_create_info.blendConstants[0] = 0.0f;
color_blending_create_info.blendConstants[1] = 0.0f;
color_blending_create_info.blendConstants[2] = 0.0f;
color_blending_create_info.blendConstants[3] = 0.0f;
```

Avec : 
- `VkBool32 logicOpEnable` : Si le blend bitwise est activé ou non
- `VkLogicOp logicOp` : Quel type d'opération bitwise on doit utiliser
- `uint32_t attachmentCount` : nombre d'`Attachment`
- `const VkPipelineColorBlendAttachmentState *pAttachments` : Pointeur vers un `Array` d'`Attachment`

## Pipeline Layout

Le `Pipeline Layout` sert à définir quelles valeurs `uniform` seront passées au runtime (matrices, textures etc.). 

```cpp
VkPipelineLayoutCreateInfo pipeline_layout_create_info{};
pipeline_layout_create_info.sType = VK_STRUCTURE_TYPE_PIPELINE_LAYOUT_CREATE_INFO;
pipeline_layout_create_info.setLayoutCount = 0;
pipeline_layout_create_info.pSetLayouts = nullptr;
pipeline_layout_create_info.pushConstantRangeCount = 0;
pipeline_layout_create_info.pPushConstantRanges = nullptr;

if (vkCreatePipelineLayout(m_device, &pipeline_layout_create_info, nullptr, &m_pipeline_layout) != VK_SUCCESS) {
	throw std::runtime_error("failed to create pipeline layout");
}
```

Sous partie à développer plus tard.

On doit détruire l'handle à la fin de notre programme : 
```cpp
vkDestroyPipelineLayout(m_device, m_pipeline_layout, nullptr);
```

On peut enfin créer la [[Graphics Pipeline]] en utilisant également le [[Render Pass]]: 

```cpp
VkGraphicsPipelineCreateInfo pipeline_create_info{};
pipeline_create_info.sType = VK_STRUCTURE_TYPE_GRAPHICS_PIPELINE_CREATE_INFO;
pipeline_create_info.stageCount = 2;
pipeline_create_info.pStages = shader_stages;
pipeline_create_info.pVertexInputState = &vertex_input_info;
pipeline_create_info.pInputAssemblyState = &input_assembly_create_info;
pipeline_create_info.pViewportState = &viewport_state_create_info;
pipeline_create_info.pRasterizationState = &rasterization_create_info;
pipeline_create_info.pMultisampleState = &multisampling_create_info;
pipeline_create_info.pDepthStencilState = nullptr;
pipeline_create_info.pColorBlendState = &color_blending_create_info;
pipeline_create_info.pDynamicState = &dynamic_state_create_info;
pipeline_create_info.layout = m_pipeline_layout;
pipeline_create_info.renderPass = m_render_pass;
pipeline_create_info.subpass = 0;

vkCreateGraphicsPipelines(
	m_device,
	VK_NULL_HANDLE,
	1,
	&pipeline_create_info,
	nullptr,
	&m_graphics_pipeline
);
```

Le seul paramètre pour l'instant nécessaire à développer est `subpass` étant l'index du `Sub Pass` du [[Render Pass]] auquel cette pipeline est associée. Un [[Graphics Pipeline]] est créée pour un `Sub Pass` bien qu'un `Sub Pass` puisse être référencé par plusieurs [[Graphics Pipeline]] (par exemple une pour les opaques une autre pour les transparents).

et il doit être détruit avant le `Pipeline Layout` : 

```cpp
vkDestroyPipeline(m_device, m_graphics_pipeline, nullptr);
```