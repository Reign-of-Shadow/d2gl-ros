# Diablo 2 LoD Glide/DDraw to OpenGL Wrapper (D2GL)
[![Github All Releases](https://img.shields.io/github/downloads/bayaraa/d2gl/total.svg)](https://github.com/bayaraa/d2gl/releases) [![License: GPL v3](https://img.shields.io/badge/License-GPLv3-blue.svg)](https://github.com/bayaraa/d2gl/blob/master/LICENSE.md)

D2GL is a Diablo 2 LoD Glide/DDraw to OpenGL wrapper that translates glide/ddraw calls to OpenGL calls. This makes D2 run better on modern hardware/OS and easier to switch between fullscreen and windowed modes, and resolves certain window and cursor-related issues. Additionally, it includes several libretro shaders for upscaling to higher resolutions, and allows for in-game text to be displayed in high-definition, D2DX's motion prediction, other QOL features.

## Reign of Shadow

This fork of [bayaraa/d2gl](https://github.com/bayaraa/d2gl) (branch `ros`) runs D2GL with the Diablo II 1.10f mod
Reign of Shadow (D2SE, D2Mod, D2ExpRes 1024x768). It is v1.3.3 with the D2SE and D2ExpRes support by
[rnd2k](https://github.com/rnd2k/d2gl) (D2GL.v1.3.1.d2se; its commits without the 1.09b support and the monster name
fix, which v1.3.3 has itself) and the changes below. Later versions of the original come in with a rebase onto its
`master`.

- **Gamma:** linear until the game loads a table. With contrast 0 the game only calls `guGammaCorrectionRGB(1, 1, 1)`,
  which the D2SE build ignores, and every color of the game was black.
- **D2ExpRes.ini** is read from the game folder (it was looked up in the Windows folder, so the high resolution was
  always 1280x768 and the picture off center).
- **D2ExpRes skip:** the `jmp` that skips D2ExpRes's patches for Sven's glide3x goes to +0x20D2 in Reign of Shadow's
  build of D2ExpRes.dll (+0x20D0 in MedianXL's); without it the game crashes on start.
- **HD text, tooltips:** Reign of Shadow's RoS.dll puts a jump to its own framed text function over D2Win #10129 after
  D2GL hooked it, and its item descriptions call that function directly; D2GL now hooks that function as well. Drawing
  them itself, D2GL calls the mod's `RoS_FramedTextPosition` export first (its position fixes); an empty text only clears
  the mod's buffer.
- **HD text, other:** the same text at the same place is drawn once a frame (RoS's character screen drew each about 16
  times, the buffer for HD objects ran over: flicker), and nothing is written past that buffer. RoS.dll also points the
  call after the automap to its own function: D2GL hooks it again, so the HUD stage starts (panel texts were taken for
  map texts and hidden) and the level entry text is not drawn twice. Map texts under a panel are hidden at any size.
  Black backgrounds the game draws itself: large ones (a window), the 32x32 under buff icons, and the 75% ones (not the
  popups of the character screen). Item descriptions end at the y RoS gives, as D2Win draws them.
  REVEAL MAP (maphack.dc6) in the escape menu.
- **Options the mod has itself** are gone from the menu and not read from d2gl.ini: No Pickup, Show Monster Resistances
  (they showed 0% with the 1.10 client), Show Item Quantity (it hooks the game's inventory loop, which RoS replaces;
  RoS.dll has the option now). Also gone: the button to the configuration wiki. The menu says "Reign of Shadow edition".
- **Minimap** can be switched on again with D2ExpRes (it drew the automap over the whole screen at first; no problem seen since, waiting for feedback of players).
- **Clock and FPS** with the automap texts at the top right (level name, version, difficulty ...), with and without HD
  text: the clock behind the first line, the FPS counter (Show FPS) as a line below; without the automap only the clock
  and the FPS below it. With HD text and the minimap, the minimap places these texts as before.

Build `glide3x` as `Release|Win32` with Visual Studio 2022 and the toolset **MSVC 14.36** (component
"MSVC v143 - VS 2022 C++ x64/x86 build tools (v14.36-17.6)"): `d2gl/vendor/lib/Release/glslang.lib` was built with
it and links only with the same compiler.

```
MSBuild glide3x\glide3x.vcxproj /p:Configuration=Release /p:Platform=Win32 /p:VCToolsVersion=14.36.32532
```

The DLL is `glide3x\bin\Release\glide3x.dll`; it goes into the mod folder with `d2gl.mpq` of the v1.3.3 release of the original.

### 🚩 Bug/Crash Report (***Important!***)

Please don't just say it crashes. I can't do anything without additional information!  
If you want to report bug or crash please include details about your system (OS, hardware), what you doing when crash occured, are other mods included (like plugy, basemod etc), is crashing on both wrappers (glide, ddraw).  
Also run game.exe with `-3dfx -log` it will create `d2gl.log` and attach this along with diablo2's log file like this `D2230410.txt`. It helps investigation.

## Features

- In-Game Option Menu(`Ctrl+O`) to change settings on fly.
- Upscale to higher resolution using RetroArch's slang shaders.
- Switch some parts in game to high-definition (all text, cursor etc.).
- Unlocked FPS (menu screen 45fps / ingame unlocked).
- Modified version of "D2DX's Unit/Weather Motion Predictor" feature (better with v-sync on).
- Few graphic post processing effects (Lut, sharpen, FXAA etc.).
- Fast switching windowed to fullscreen with `Alt+Enter` key.

## Requirements

- Diablo 2 LoD: ``1.09d``, ``1.10f``, ``1.11``, ``1.11b``, ``1.12a``, ``1.13c``, ``1.13d``, ``1.14d``.
- Windows 7/8/10/11, Linux with Wine (Proton, Lutris etc), MacOS with Wine (Crossover).
- GPU with minimum OpenGL 3.3 support.

## Installation

See [Installation](https://github.com/bayaraa/d2gl/wiki/Installation).

## Configuration

See [Configuration](https://github.com/bayaraa/d2gl/wiki/Configuration).

## Compatibility

See [Compatibility](https://github.com/bayaraa/d2gl/wiki/Compatibility).

## Credits

Diablo II modding community (The Phrozen Keep) and **Everyone** who makes d2 mod and their source codes available.

- SGD2FreeRes D2 Custom Resolution (Mir Drualga).
- D2DX's Unit/Weather Motion Predictor (Bolrog).
- Libretro's slang shaders (RetroArch).
- The OpenGL Extension Wrangler Library (Brian Paul).
- OpenGL Mathematics (GLM) (G-Truc Creation).
- FXAA implementation by Timothy Lottes (NVIDIA).
- Dear ImGui (Omar Cornut).
- stb_image, stb_image_write (Sean Barrett).
- MurmurHash3 (Austin Appleby).
- MSDF Atlas Generator (Chlumsky).
- Shader Minifier (laurentlb).
- SPIRV-Cross, glslang (KhronosGroup).
- Detours (Microsoft).

## Some Screenshots

![Screenshot010](https://user-images.githubusercontent.com/2043880/220664490-2a9b34d8-ca7c-4e52-a57d-d43c508f5813.png)
![Screenshot012](https://user-images.githubusercontent.com/2043880/220668775-3351be3b-27fa-4800-883f-09e5eb935c47.png)
![Screenshot013](https://user-images.githubusercontent.com/2043880/220666692-967a8c13-f480-4ac6-af1e-b45fda3bdee3.png)
![Screenshot002](https://user-images.githubusercontent.com/2043880/220667272-a83aa2cd-d038-41ea-a878-a6e148b8f9f6.png)
