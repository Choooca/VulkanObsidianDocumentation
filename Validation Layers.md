L'API Vulkan vise à minimiser l'overhead au maximum, au point que la vérification d'erreurs est minimale par défaut. Pour pallier ça Vulkan met à disposition les [[Validation Layers]] qui sont des composants optionnels qui interceptent les appels Vulkan pour effectuer des vérifications supplémentaires et remonter les erreurs. Elles servent notamment à : 

- Vérifier les paramètres des fonctions pour détecter s'il y a des erreurs
- Tracker les ressources pour prévenir de leaks
- Check la thread safety 
- etc.

Vulkan ne possède pas de [[Validation Layers]] natives, mais des SDK comme [LunarG Vulkan SDK](https://vulkan.lunarg.com/home/welcome) mettent à disposition de nombreuses [[Validation Layers]] utiles.
Cela signifie que tous les systèmes ne possèdent pas les [[Validation Layers]] utilisées dans notre application. Nous devons donc d'abord vérifier les [[Validation Layers]] disponibles avant de les activer. Pour cela : 

```cpp
uint32_t layer_count; 
vkEnumerateInstanceLayerProperties(&layer_count, nullptr); 

std::vector<VkLayerProperties> available_layers(layer_count);
vkEnumerateInstanceLayerProperties(&layer_count, available_layers.data());
```

Nous pouvons alors comparer les `available_layer.layerName` avec les layers que nous souhaitons pour vérifier la compatibilité des layers demandées avec les layer disponibles.

Les messages s'écrivent par défaut dans le standard output. Cependant nous pouvons setup des [[Message Callback]] permettant de gérer manuellement l'ouput afin de par exemple trier les messages que nous voulons afficher.

Ressources : 
[Vulkan Tutorial](https://vulkan-tutorial.com/Drawing_a_triangle/Setup/Validation_layers)