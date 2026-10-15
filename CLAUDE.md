# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

Fork of yujitach/MenuMeters: a macOS menu bar system monitor (CPU, disk, memory, network). Pure Objective-C, ARC, GPL v2. No tests, no formatter, no CI beyond a stale `.travis.yml`.

## Build

Unsigned build (the project pins a `DEVELOPMENT_TEAM` that isn't ours, so always override signing):

```
xcodebuild -project MenuMeters.xcodeproj -scheme "MenuMeters No Sparkle" -configuration Release build CODE_SIGN_IDENTITY='-' DEVELOPMENT_TEAM=''
```

- Schemes: `MenuMeters` (with Sparkle) and `MenuMeters No Sparkle`. Prefer the latter for verification.
- Bare `xcodebuild` builds Release; there is no test command.

## Architecture

- Faceless app (`LSUIElement`). `MenuMetersApp/AppDelegate.m` instantiates the four extras (`MenuExtras/MenuMeter{CPU,Disk,Mem,Net}/`) directly as `NSStatusItem`s, all subclassing `MenuMetersMenuExtraBase` in the repo root. They are not SystemUIServer `NSMenuExtra`s despite the names.
- `PrefPane/` is the old System Preferences pane code, now embedded in the app (`MenuMetersPref`). Preferences live in `Common/MenuMeterDefaults`.
- `hardware_reader/` holds the SMC / Apple Silicon temperature readers (IOKit).
- `MenuMeters.pch` already imports Cocoa, Carbon, SystemConfiguration, IOKit and Security — don't add those imports.

## Gotchas

- All Sparkle code is behind `#ifdef SPARKLE`; only the `MenuMeters` target defines `SPARKLE=1`. `Sparkle.framework` is vendored in the repo root.
- Version is `MM_VERSION` in `InfoPlistPreprocessor.h`, not a plist literal.
- UI strings are localized in both `MenuMetersApp/` and `PrefPane/` (`.lproj` for de, en, fi, fr, it, ja, nl, tr, zh-Hans); update every locale when adding a key. `baselocalizationfixes/` is a historical migration toolkit, not part of the build.
- Code style: match the surrounding file (older files use tabs and a GPL header, newer ones 4 spaces).

## Workflow

- Work on a feature branch and open a PR into `75033us/MenuMeters` `main`; don't push to `main` directly.
- Don't touch `releases.html` or the Sparkle feed URL unless the task is a release.
