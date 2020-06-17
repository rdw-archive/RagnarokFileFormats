# Fog Effects

This document details the findings of my research into the rendering of fog effects by Gravity's client. All information was derived by decompilation/disassembly, using a 2008 client with provided debug information.

I don't believe this particular facet has changed since then. The effects were hardcoded, they seemingly didn't upgrade to a different DirectX version, and there aren't any Lua files referring to fog effects in more recent clients either. So the natural conclusion is that they left everything the way it was, though I haven't confirmed this yet.

## Fog Parameters Table

* fogparameterstable.txt
* Still present in modern clients
* No Lua equivalent as of 2013?

**Check 2020 version also to be sure? Should have cloud tga's hardcoded still, and no Lua for clouds/fog**

## Fog Modes

**Does the client support multiple fog modes?**

The client (**only theoretically?**) supports all fog effect modes offered by the DirectX7 APIs. These are:

* ``D3DFOG_NONE``: No visible fog/fog is disabled (**Toggled via /fog command?**)
* ``D3DFOG_EXP``: Fog that exponentially increases in density  (**Confirm it behaves the same way as it does in WebGL?**)
* ``D3DFOG_EXP2``: Similar, but increases even faster?  (**Confirm it behaves the same way as it does in WebGL?**)
* ``D3DFOG_LINEAR``: The density increases uniformly, but is based on the distance to a near and far plane (start and end of the foggy area)? (**Confirm it behaves the same way as it does in WebGL?**)
* ``D3DFOG_FORCE_DWORD``: I have no idea? (**Research?**)

**Which fog modes are actually used? When?**

**(fogMode = 3 is set in the SwitchFog function, so it would be linear?)**

**Does it use the alpha (AARRGGBB) from fogparamtable?**

*** Does it use the last parameter at all? (density?)**

# Fog Textures

There are some conspicuously-named image files located inside the ``data.grf``:

*  ``fog1.tga``
*  ``fog2.tga``
*  ``fog3.tga``

However, contrary to what one might think, these are instead used to render cloud effects. Fog is actually generated using the Direct3D API, which provides a simple method and doesn't require (**or even support?**) using textures to create fog effects.

## Sources

* ``CRagEffect::Cloud``


Relevant variables:

* g_fogPara.mode = see above
* g_fogPara.useRange = ??
* g_fogPara.density = only EXP/EXP2? (seems unused in LINEAR mode)
* g_fogPara.end = far limit
* g_fogPara.start = near limit
* this->m_isFoggy
* g_session.m_fogOn: Determines whether to create (MakeFog) the fog when the session is started?

**What units do the near/far limits use? 10x scale like GND/RSW? How does it relate to hardcoded values for FAR_LIMIT in RJS/Aesir? (1200/240)?**

Relevant functions:

* RPLmFace::InitSpecular(void) = ??
* CRenderer::FogSwitch(int) = toggle fog effect? seems to only use mode 3 (linear)
* CRenderer::SetLight(struct vector3d &,struct vector3d &,struct vector3d &) = unrelated?
* CRendererer->m_isFoggy = is fog currently visible?
* CRendererer->m_isVertexFog = ?? seems to be some low level/D3D stuff, with line primitives? Can't tell what it does, yet
* InitFogParameterTable = Read fog parameters from text file (hardcoded path)


## TODO

* Open questions (bold) require more research
* Provide all sources
* Links to the "fog" textures
* Link to ``CRagEffect``
* Link to the ``g_fogPara`` struct
* Link to DX7 APIs to create Fog effects (D3D7?)
* Link to the ``_D3DFOGMODE`` enum