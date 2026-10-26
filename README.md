# DoubleExposureTweaks

A ReShade add-on for **Life is Strange: Double Exposure** (Unreal Engine 5.2, DirectX 12) that raises
lighting, shadow and detail settings beyond the game's menu, live, from the ReShade overlay. Built
from ReunionTweaks.

Work in progress (0.1.1): the options aren't tested in this game yet.

## Install

Needs ReShade 6.8 or newer **with add-on support**, installed for the game's
`Chronos\Binaries\Win64\Chronos-Win64-Shipping.exe` (DirectX 12). Copy `DoubleExposureTweaks.addon64`
next to that exe and open the ReShade overlay (Home) in game.

## Use

- Pick a quality preset or set options per tab; changes apply at once and are saved in ReShade.ini.
- **F8** switches all tweaks off and on to compare.
- **Advanced** tab: status, a hardware ray tracing test, a probe that writes every relevant
  setting to a file, and a console variable field for experiments.

### Hardware ray tracing test

Ray tracing can only be switched on at startup, so **Test on next launch** writes `r.RayTracing=1`
(and the skin cache setting ray tracing needs) to your `Engine.ini` in
`%LOCALAPPDATA%\Chronos\Saved\Config\Windows\`. Restart the game: the add-on reports whether it
started with ray tracing on. If the game stops at start instead, start it again; the add-on sees
the test never finished and takes the setting out.

## Files it writes (next to the game exe)

- `DoubleExposureTweaks.log`
- `DoubleExposureTweaks-probe.txt` (setting values, when probed)
- `DoubleExposureTweaks-rttest.ini` (ray tracing test state)

## Build

`.\build.ps1 [-Install]` with Visual Studio 2022+ (C++ x64 tools). Details in `docs/FINDINGS.md`.
