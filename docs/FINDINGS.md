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

## Hardware ray tracing: untested

`r.RayTracing`, `r.RayTracing.Enable`, `r.Lumen.HardwareRayTracing`, `r.Lumen.Reflections.HardwareRayTracing`,
`r.RayTracing.Shadows/Reflections/AmbientOcclusion/Translucency`, `r.SkinCache.CompileShaders` all
exist. Whether the ray tracing shaders were cooked can only be told by starting the game with
`r.RayTracing=1` (Reunion stopped at start: "Missing global shader FNiagaraCollisionRayTraceMiss").
The add-on's Advanced tab runs that test (see main.cpp, "ray tracing test").

## Still to do

- First launch: check the log's layout lines (IsVariableBool slot, data/ref copy, sinks flag) and
  the probe (`DoubleExposureTweaks-probe.txt`) for the menu's values per tier.
- Run the ray tracing test.
- Measure preset costs.
- DLSS (separate add-on, from ReunionDLSS): TSR passes must be captured again (5.2's TSR differs).
