Contrairement à d'autres API, Vulkan n'utilise pas directement du GLSL ou du HLSL, mais du byte code. Le format de ce byte code est appelé `SPIR-V`. Cela permet d'avoir une conversion moins complexe entre le shader code et le code machine, et d'éviter les divergences d'interprétation entre les différents drivers GPU.

Khronos fournit un compilateur pour convertir nos shaders en byte code `SPIR-V`. J'utilise personnellement le compilateur de Google glslc car il utilise les mêmes formats de paramètre que d'autres compilateurs et permet des fonctionnalités comme les #includes. Les compilateurs sont directement inclus dans le SDK.

Sur Windows on peut compiler le code en utilisant un .bat de cette manière : 
```bash
C:/VulkanSDK/x.x.x.x/Bin/glslc.exe shader.vert -o vert.spv
C:/VulkanSDK/x.x.x.x/Bin/glslc.exe shader.frag -o frag.spv
pause
```

On peut également inclure le compilateur via une librairie pour compiler les shader au runtime.

Une fois nos shaders compilés on peut les lire pour les intégrer dans notre pipeline. On récupère dans un premier temps le code sous forme de `std::vector<char>` :

```cpp
std::vector<char> ReadFile(const std::string& file_name) {
	std::ifstream file(file_name, std::ios::ate | std::ios::binary);

	if(!file.is_open()) {
		throw std::runtime_error("failed to open file: " + file_name);
	}

	size_t file_size = (size_t)file.tellg();
	std::vector<char> buffer(file_size);

	file.seekg(0);
	file.read(buffer.data(), file_size);

	file.close();

	return buffer;
}
```

Nous pouvons ensuite créer les [[Shader Modules]] :
```cpp
VkShaderModuleCreateInfo create_info{};
create_info.sType = VK_STRUCTURE_TYPE_SHADER_MODULE_CREATE_INFO;
create_info.codeSize = code.size();
create_info.pCode = reinterpret_cast<const uint32_t*>(code.data());

VkShaderModule shader_module;
vkCreateShaderModule(m_device, &create_info, nullptr, &shader_module)

```

Avec `VkShaderModuleCreateInfo` : 
- `VkStructureType sType` : Le type de la structure
- `size_t codeSize` : Taille du code en bytes
- `const uint32_t *pCode` : Pointeur vers notre code

Et Pour créer le [[Shader Modules]] :
- `VkDevice device` : Notre [[VkDevice]]
- `const VkShaderModuleCreateInfo *pCreateInfo` : structure remplie avant
- `const VkAllocationCallbacks *pAllocator` : Custom allocator
- `VkShaderModule *pShaderModule` : Pointeur vers le [[Shader Modules]]. Valeur de retour

Et on peut détruire les [[Shader Modules]] à la fin de la création de la [[Graphics Pipeline]] : 
```cpp
vkDestroyShaderModule(m_device, vert_shader_module, nullptr);
```

