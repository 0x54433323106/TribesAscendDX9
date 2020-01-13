# TribesAscend_DX_TA

Hooking DX9/D3D9 in Tribes Ascend. The hooking functionality should be portable to be any game using DX9.

# DX9 frame call structure
1. BeginScene
2. Draw(IndexedPrimitive, Primitive, etc)
3. EndScene
4. Present

# Hook process
VMT hooking is idealised, we want to use only Jump (global) hooks for capturing all objects calling the specified member function.
Each object has its own VMT so a VMT hook is only local to an object.
## Finding LPDIRECT3DDEVICE9 device
1. Create a new device and jump hook the endScene function in the VMT of the newly create device. Since all devices will call the same endScene function (assuming the function isn't already hooked in their VMTs) this allows us to capture the main device used by the game itself.
2. In the endScene jump hook function, save the device parameter as a global var named *mainDevice* which is the device the game is actually using.
3. Unhook the jump hook and set up VMT hooks on the main device

## VMT hooking functions
The DirectX SDK is some sort of C implementation (not C++) that tries to emulate C++ objects. In usual C++, member functions pass a *this* pointer of the object through the *ECX* register using the *THISCALL* calling convention. In this case however all functions called from a DirectX "object" pass a pointer of itself as the first argument. This narrows the calling convention down to either *CDECL* or *STDCALL*, with the latter being the case.

So to hook "member functions" we must use **__stdcall** and (almost) always set the first parameter of hook function as the "object" (ie. Sprite, Device, Font, etc.). This is to ensure the hook function mathces the signature of the member function being hooked.

For example, the hook function signature for Reset is
`HRESULT __stdcall resetHook(LPDIRECT3DDEVICE9 device, D3DPRESENT_PARAMETERS* pPresentationParameters)`
and to call the original function we can do something like
```
typedef HRESULT(__stdcall* reset)(LPDIRECT3DDEVICE9, D3DPRESENT_PARAMETERS*);
...
HRESULT __stdcall resetHook(LPDIRECT3DDEVICE9 device, D3DPRESENT_PARAMETERS* pPresentationParameters) {
    HRESULT res = ((reset)resetVMTHook->getOriginaFunction())(device, pPresentationParameters);
    return res;
}
```

The exact definitions of the functions can be found from the online reference or looking at the SDK header files.

## Device functions
There are numerous device functions that we need to hook to do anything useful.
* Reset - This needs to be hooked to successfully handle alt tabbing and resizing.
* EndScene - The function itself provides nothing useful but it's a place where we can write code to do something at the end of each frame
* Present - Same thing as endScene.
* DrawIndexedPrimitive - Most primitives are drawn through this function. Hooking it allows us to directly manipulate primitive states and textures.
* SetTexture - Hooking this gives access to all the textures being set in a frame. We can save them from here or through DrawIndexPrimitive.

# Quirks and tidbits
1. Mid function hooking Present (at a sufficiently far offset) allows us to have an "OBS bypass" (possibly may be able to work for other recording software too). So far I've only managed to execute basic drawing code in here but anything drawn in this function will be entirely hidden from the OBS recording. So we can render the ImGui menu in here, or a crosshair, custom HUD bars, or even TAMods route markers.
![Image](https://cdn.discordapp.com/attachments/596698457532792833/596698619445248011/MidFunctionPresent.jpg)

Notice how the crosshair, HUD bars and route markers are completely omitted from the OBS display.

## References
https://docs.microsoft.com/en-us/windows/win32/api/_direct3d9/

https://docs.microsoft.com/en-us/windows/win32/api/d3d9/nf-d3d9-idirect3ddevice9-present