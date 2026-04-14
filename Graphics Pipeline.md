
La [[Graphics Pipeline]] est la séquence d'opérations transformant les données d'entrée (vertices, indices, textures etc.) en pixels écrits dans un [[Framebuffer]]

![[vulkan_simplified_pipeline.jpg]]

Dans la [[Graphics Pipeline]] il y a deux type d'étape :

- Les fixed-functions, ce sont des étapes, comme le `Color blending`, qui sont prédéfinies mais qu'on peut configurer à l'aide de paramètres. Elles sont notées en verte sur le schéma.
- Les programmable, ce sont les étapes, comme le `Fragment shader`, où l'utilisateur peut upload son propre code et donc programmer son propre comportement. Elles sont notées en orange sur le schéma.