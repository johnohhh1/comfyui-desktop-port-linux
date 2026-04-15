<p align="center">
  <img src="assets/UI/Comfy_Icon_Windows.png" alt="Comfy UI" width="600"/>
</p>

# ComfyUI Desktop for Linux

[![License](https://img.shields.io/badge/license-GPL--3.0-blue.svg)](https://www.gnu.org/licenses/gpl-3.0.en.html)
[![Platform](https://img.shields.io/badge/platform-Linux%20%7C%20Ubuntu%2026.04-orange.svg)](#build-from-source)
[![Version](https://img.shields.io/badge/version-0.8.30-brightgreen.svg)](#quick-start)
[![Upstream](https://img.shields.io/badge/upstream-Comfy--Org%2Fdesktop-informational.svg)](https://github.com/Comfy-Org/desktop)

An Ubuntu-focused Linux port of the official ComfyUI Desktop app.

This fork keeps the upstream Electron desktop shell, bundled frontend, asset bootstrap, and packaging flow, then patches the parts that blocked Linux so the app can actually build and run on Ubuntu 26.04.

## What This Is

- A Linux build of the ComfyUI Desktop app
- Packaged as a `.deb`
- Built from the upstream `Comfy-Org/desktop` source
- Patched to work on Ubuntu 26.04

It is not a web wrapper and it is not a fake launcher. It is the desktop app, ported.

## What Works

- Linux build and packaging
- Bundled `desktop-ui` assets in packaged builds
- `.deb` installation on Ubuntu 26.04
- Linux config/reset path handling
- Linux-safe startup behavior for Electron
- Asset bootstrap on Linux via `yarn make:assets`
- Existing local ComfyUI detection
- Automatic attach to an already-running local ComfyUI server on `127.0.0.1:8188`

## Linux Behavior

This fork supports two practical flows.

### 1. Existing ComfyUI install

If both of these are true:

- `~/ComfyUI` exists and looks like a valid ComfyUI directory
- a local ComfyUI server is already running on `127.0.0.1:8188`

then the desktop app will:

- detect that install
- persist it as the desktop `basePath`
- switch into external-server mode
- skip the desktop-managed Python bootstrap path
- open against the running ComfyUI instance instead of forcing a fresh install

This is the mode you want if you already run ComfyUI in the browser and just want the desktop shell.

### 2. Fresh desktop-managed install

If no compatible local install/server is detected, the app falls back to the normal desktop onboarding flow and can set up its own managed install path.

## Why `.deb` Instead of `AppImage`

The upstream app can be built as an AppImage, but in practice Electron on Linux hits the usual `chrome-sandbox` failure inside the mounted AppImage runtime.

For Ubuntu, `.deb` is the cleaner target:

- no AppImage sandbox helper nonsense
- normal desktop integration
- normal installation path under `/opt/ComfyUI`
- normal launcher registration

## Quick Start

### Install the built package

```bash
sudo apt install ./dist/ComfyUI-0.8.30-amd64.deb
```

Or install from an absolute path:

```bash
sudo apt install /path/to/ComfyUI-0.8.30-amd64.deb
```

### Launch

Use your desktop launcher or run:

```bash
/opt/ComfyUI/@comfyorgcomfyui-electron --no-sandbox --disable-gpu
```

`--disable-gpu` here affects Electron rendering, not PyTorch/CUDA inference inside your ComfyUI server.

## Build From Source

### Requirements

- Ubuntu 26.04
- Node 20
- `corepack`
- Python 3.12+
- system libraries required by Electron packaging

Useful packages:

```bash
sudo apt install build-essential git curl libgtk-3-dev libnotify-dev libnss3-dev libxss-dev libxtst-dev libatspi2.0-dev libsecret-1-dev
```

### Install dependencies

```bash
corepack enable
corepack yarn install
```

### Bootstrap assets

```bash
corepack yarn make:assets
```

This pulls in the bundled ComfyUI source, frontend assets, and Linux `uv` artifacts used by the desktop app.

### Verify the codebase

```bash
corepack yarn typecheck
corepack yarn vitest run
```

### Build the package

```bash
corepack yarn make
```

That produces:

```bash
dist/ComfyUI-0.8.30-amd64.deb
```

## Key Linux Patches In This Fork

### Build and packaging

- enabled Linux asset bootstrap in `scripts/preMake.js`
- enabled Linux verification in `scripts/verifyBuild.js`
- packaged `desktop-ui` into Linux builds
- switched Linux target to `.deb` in `config/builder-debug.config.ts`

### Runtime

- added Linux config-reset handling in `scripts/resetInstall.js`
- made Linux hardware validation non-fatal in `src/utils.ts`
- added fallback when compiled requirements are not present in `src/virtualEnvironment.ts`
- added Linux-safe Electron startup behavior in `src/main.ts`

### Existing install / external server support

- default install path now prefers an existing `~/ComfyUI`
- startup now detects a running local server on `127.0.0.1:8188`
- persisted external-server mode through desktop config
- skips desktop-managed venv validation when attaching to an existing live server

## Paths

### Installed app

```text
/opt/ComfyUI
```

### Desktop launcher

```text
/usr/share/applications/@comfyorgcomfyui-electron.desktop
```

### Desktop config

```text
~/.config/ComfyUI/config.json
```

### Logs

```text
~/.config/ComfyUI/logs
```

## Known Caveats

- This is an Ubuntu-first Linux port, not a heavily tested multi-distro release.
- The upstream desktop app still assumes Windows/macOS in a few places; this fork patches the important ones, not every cosmetic assumption.
- External-server auto-attach currently targets the common local setup:
  - install path: `~/ComfyUI`
  - server: `127.0.0.1:8188`
- If you run ComfyUI somewhere else, you can still patch the config or source to match your layout.

## Upstream

- App source: https://github.com/Comfy-Org/desktop
- ComfyUI core: https://github.com/comfyanonymous/ComfyUI
- ComfyUI frontend: https://github.com/Comfy-Org/ComfyUI_frontend

## License

This fork remains under the upstream project license:

`GPL-3.0-only`

