# DoubleExposureTweaks 1.0.0

Graphics add-on for **Life is Strange: Double Exposure** (PC, Steam), for ReShade with add-on
support. Raises the game's lighting, shadow, character and detail settings beyond what its menu
offers, live, from an in-game overlay.

- **Lighting** (software Lumen): bounce light traced against each object instead of a coarse
  whole-scene approximation (about 5% here), more lighting probes, reflections on rougher
  surfaces and on glass and water.
- **Shadows**: sharper lamp shadows (the game keeps them softer than the sun's), smoother soft
  shadows, longer shadow distance, full-resolution character soft shadows.
- **Characters**: full-resolution skin shading (the game uses a checkerboard), higher-quality
  strand hair.
- **Detail**: texture memory beyond the game's fixed 1.4 GB, 16x anisotropic filtering, draw
  distance, finer geometry, finer volumetric fog, higher-precision TSR history.
- **Post-processing**: switches for film grain, lens flares, depth of field, motion blur and
  chromatic aberration.
- **Quality presets**: *Ultra+* (everything beyond the menu that costs little; about 6-7% slower)
  and *Cinematic* (the costlier upgrades that still play well; about 30% slower). Measured on an
  RTX 5080 at 4K with 70% resolution scale in one scene. **Measure preset costs** runs each one
  for a few seconds and shows its frame rate on your PC; **Measure each option** ranks every
  option by cost (the most expensive choices, such as Cinematic lighting probes, are left out of
  the presets and marked in their tooltips).
- **F8** switches every tweak off and on, to compare.

The game has no hardware ray tracing: its ray tracing shaders weren't shipped (starting it with
`r.RayTracing=1` stops it at launch), so nothing here uses it. The Advanced tab has a test that
checks this safely on your copy. Every option starts at the game's own value, and its tooltip
says what the game uses and what the choice costs.

## Requirements

- Life is Strange: Double Exposure for PC (Steam build 16505703 tested; Unreal Engine 5.2). If
  the game's code doesn't match, the add-on changes nothing and says so in the overlay and in
  `DoubleExposureTweaks.log`.
- [ReShade](https://reshade.me) 6.8 or newer **with full add-on support**, installed for
  `Chronos-Win64-Shipping.exe` with DirectX 10/11/12.

## Install

1. Install ReShade with add-on support into
   `...\LifeIsStrangeDoubleExposure\Chronos\Binaries\Win64`.
2. Copy `DoubleExposureTweaks.addon64` into the same folder.
3. Start the game and press **Home** to open ReShade. The settings are on the **Add-ons** tab,
   under DoubleExposureTweaks.

Settings are saved in `ReShade.ini` under `[DoubleExposureTweaks]`. No game code is patched and,
apart from the log and the optional settings dump, nothing else is written to disk; the one
exception is the ray tracing test, which writes `r.RayTracing=1` to your `Engine.ini` for one
launch when you start it and takes it out again (even if that launch fails). Deleting the
`.addon64` file removes the mod completely.

## Advanced tab

- **Hardware ray tracing test**: writes the setting for the next launch, then reports whether
  the game started with ray tracing on. The test launch removes the setting as soon as it starts,
  so if the game stops at launch, the next start is normal.
- **Probe**: writes every lighting, shadow, upscaling and detail setting with the game's value to
  `DoubleExposureTweaks-probe.txt`.
- **Console variable**: reads and sets a single setting, for experiments.

## Building

`.\build.ps1` builds `bin\DoubleExposureTweaks.addon64` (needs Visual Studio 2022 with the C++
tools). `.\build.ps1 -Install` also copies it into the game folder (close the game first);
`.\build.ps1 -Package` makes the release zip in `bin\release`. See `docs/FINDINGS.md` for how the
game's settings were found (UE 5.2's console variable layout differs from 5.5/5.6).

## License

MIT (see `LICENSE`). Third-party headers: see `THIRD_PARTY_NOTICES.txt`.
