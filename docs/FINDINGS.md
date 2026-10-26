# Life is Strange: Double Exposure: findings

Game: Life is Strange: Double Exposure (Deck Nine), Steam app 1874000, build 16505703. Unreal Engine
**5.2.1** (project `Chronos`; the crash reports say `5.2.1-0+UE5`, build tree
`D:\Chronos-Test-Prog\UE5`). DirectX 12 (Agility SDK `D3D12Core.dll` shipped).

Paths:
- Game: `E:\SteamLibrary\steamapps\common\LifeIsStrangeDoubleExposure`
- Executable: `Chronos\Binaries\Win64\Chronos-Win64-Shipping.exe` (134 MB; `LifeIsStrangeDoubleExposure.exe`
  in the root is a launcher stub)
- User config: `%LOCALAPPDATA%\Chronos\Saved\Config\Windows\` (`Engine.ini`, `GameUserSettings.ini`).
  No per-Steam-user subfolder, unlike Reunion.
- ReShade 6.8.0 (add-on build) `dxgi.dll` was already in the exe folder (2026-08-02).

The user's earlier read-only Engine.ini (2026-08-20: HDR output cvars, DOF / motion blur /
chromatic aberration off, `r.LUT.UpdateEveryFrame=1`) was renamed to
`Engine.ini.custom-2026-08-20.bak` on 2026-10-26 at their request; the game makes a fresh one.
Menu settings at that time: 3840x2160, HDR on at 1000 nits, scalability mostly 2-3.

Two crash reports from 2026-08-06: `LowLevelFatalError` in `D3D12Util.cpp` line 873 (a failed D3D12
call), before any of this work.

## Not protected

Normal sections (`.text` R X, `.rdata` R, `.data` RW, `.pdata`, `_RDATA`, `.rsrc`, `.reloc`): no
anti-tamper, so code could be patched if ever needed (Reunion's couldn't).

## No upscaler plugins

Only TSR. No DLSS, NGX, Streamline or XeSS strings. The menu's string table has `SETTINGS_USEFSR`
and the ASCII text `r.FidelityFX.FSR3.Enabled`, but no cvar of that name is registered (no UTF-16
name in the exe), so the menu's FSR switch most likely does nothing. Deck Nine's secondary
resolution: `D9.Rendering.TargetSecondaryResolutionVertical` and `r.SecondaryScreenPercentage.GameViewport`.

## Console variable layout

Read from the game's own call sites (scratch scripts; the add-on checks it again at run time):

- `IConsoleManager::Singleton` at RVA `0x79c6750` (every `r.ScreenPercentage` lookup loads it).
- `FindConsoleVariable` is `+0xb8` (as in UE 5.5; 4.27 had `+0xb0`): 1485 manager calls.
- After it: `+0x58` AsVariableInt (536), `+0x90` GetInt (50), `+0x60` AsVariableFloat (37), `+0x88`
  GetBool (16), `+0x98` GetFloat. `mov r8d, SetBy` is followed by `call [rax+0x80]` (Set).
- So `IConsoleVariable` is laid out as in UE 4.27 (AITDTweaks): IsVariable* `0x30-0x48`, AsVariable*
  `0x50-0x68`, Set `0x80`, GetBool/Int/Float `0x88/0x90/0x98`. `CallAllConsoleVariableSinks` is
  expected at manager `+0x50` (12 call sites).
- SetBy values passed in the exe: `0x00` (324), `0x01` 21, `0x02` 23, `0x03` 7, `0x04` 11, `0x05` 3,
  `0x08` 11, **`0x09` 51 (SetByCode)**, **`0x0A` 11 (SetByConsole)**, nothing higher. The add-on sets at
  `0x0A`.
- UE 5.2 has no `ECVF_Set_SetOnly_Unsafe`. Set copies the value to the render thread only on the game
  thread, so the add-on copies it itself (`cvars::MirrorToRenderer`: data cvars via AsVariable*, Ref
  cvars via their getter's code). Giving a value back to the game puts its old priority back so the
  menu can change it again.

Game-specific cvars: `D9.*` (40), among them `D9.Rendering.TargetSecondaryResolutionVertical`,
`D9.UseAdvancedColorGrading(.BlurSize)`, `D9.UseAdvancedColorStrokes`, `D9.EnableShadowCastingPriority`,
`D9.ForceArcadeQuality`, `D9.EmulateLowEnd`, `D9.DoubleExposureBrightness`, `D9.BasePhotoBrightness`.
No `D9.SigMan.*` (Reunion's character detail manager).

Not in 5.2 (dropped from ReunionTweaks' options): `r.Lumen.Reflections.MaxRoughnessToTraceClamp`,
`r.Shadow.Virtual.ResolutionLodBias*Moving`, `D9.SigMan.*`.

## No hardware ray tracing

`r.RayTracing`, `r.Lumen.HardwareRayTracing` and the other ray tracing cvars exist, but the project
has `r.RayTracing=0` (set by `0x03`, project settings) and no ray tracing shaders were cooked. The
add-on's test (2026-10-26) started the game with `r.RayTracing=1` in Engine.ini
(`r.SkinCache.CompileShaders` was already 1) and it stopped at start:

    LowLevelFatalError [ShaderCompiler.cpp] [Line: 7173]
    Missing global shader FVisualizeSkyLightMipTreePS's permutation 0, Please make sure cooking was successful.

Same as Reunion. Lumen always runs in software; nothing in the add-on uses hardware ray tracing.

The first version of the test only removed the setting on the launch *after* the crash, but the
engine reads Engine.ini before ReShade loads the add-on, so that launch would have crashed too.
0.1.1 takes the lines out as soon as the test launch starts and writes them back only if the
check passes.

## First probe (title screen, 5 s after connecting, 2026-10-26)

1354 values read, so the 5.2 layout holds (IsVariableBool slot found, getters work, priority bytes
sane). Menu as in `GameUserSettings.ini` (shadows, GI, reflections, post, shading at 2; the rest 3).
Values (set by):

| cvar | value |
|---|---|
| `r.Lumen.TraceMeshSDFs.Allow` | 0 (0x01) |
| `r.Lumen.ScreenProbeGather.DownsampleFactor` | 16 (0x01) |
| `r.Lumen.Reflections.MaxRoughnessToTrace` | 0.4 |
| `r.Shadow.Virtual.ResolutionLodBiasDirectional` / `Local` | -0.5 / 0.5 (0x01) |
| `r.Shadow.Virtual.SMRT.RayCountLocal` | 8 |
| `r.Shadow.Virtual.MaxPhysicalPages` | 4096 (0x09) |
| `r.Streaming.PoolSize` | 1400 (0x05, device profile) |
| `r.MaxAnisotropy` | 8 |
| `r.SSS.Checkerboard` | 2 |
| `r.HairStrands.Visibility.MSAA.SamplePerPixel` | 4 |
| `r.TSR.History.R11G11B10` | 1 |
| `r.VolumetricFog.GridPixelSize` | 8 |
| `r.Nanite.MaxPixelsPerEdge` | 1 |
| `r.ScreenPercentage` | 100 (0x09); `r.SecondaryScreenPercentage.GameViewport` 100 (0x02) |
| `D9.Rendering.TargetSecondaryResolutionVertical` | 1080 |
| `r.AntiAliasingMethod` | 4 (TSR) |
| `r.HDR.Display.MaxLuminance` / `OutputDevice` | 1000 / 3 |
| `r.FilmGrain` | 1 |

No 60 s probe yet (the launch was closed sooner), and setting values hasn't been tried yet.

## Setting values works (2026-10-26)

Run-time layout check, as predicted offline: IsVariableBool at `+0x30`; `r.MaxAnisotropy` and
`r.ScreenPercentage` are data cvars with their value at `+0x48`; sinks flag at manager `+0xe0`;
singleton found by all 6 lookups. The user plays at 70% resolution scale, HDR10 PQ.

Ultra+ in a scene, F8 off/on twice: 27 values applied every time, 0 failed, 0 render-thread copy
warnings (all are data cvars), nothing changed back by the game. With tweaks off (60 s probe) every
held cvar was back at the game's value **and** its original SetBy priority (0x00/0x01/0x05), so the
menu can change them again. Only `r.Shadow.Virtual.MaxPhysicalPages` differed from the add-on's
connect-time snapshot: 6144 (0x01) at connect, then the game's code set 4096 (0x09) before Ultra+ was
picked, and 4096 is what came back.

Not shown by the log: whether the renderer uses the new values at once. Measure preset costs (a
frame-time change) or an F8 visual comparison shows that.

## Still to do

- Measure preset costs; F8 visual check.
- A probe at each menu tier.
- DLSS (separate add-on, from ReunionDLSS).
- Measure preset costs.
- DLSS (separate add-on, from ReunionDLSS): TSR passes must be captured again (5.2's TSR differs).
