# LODs, notexturestreaming and DirectX9 stages
*The information here might not be exactly correct since I'm not experienced with DirectX.*

* In DX9 objects can have up to 8 textures stages (0 to 7). Each texture stage has a texture and a blending mode, similar to layers in Photoshop.
For example, the floor of a map is not just one single texture, but multiple textures overlayed/blended ontop of each (and each texture is assigned to a texture stage).
* Each texture is assigned a *Level of detail* (LOD) and can have multiple levels. An LOD value of 0 means that the texture will be drawn at its highest level of detail. As the LOD value increases the level of detail of the texture decreases. An LOD value of -1 means that the lowest level of detail will be assigned to that texture.
* By specifying the LOD for each texture stage you can manually manipulate what LOD a texture from that stage should use.