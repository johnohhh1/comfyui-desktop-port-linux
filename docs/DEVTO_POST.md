---
title: I Ported ComfyUI Desktop to Ubuntu 26.04
published: false
tags: linux, ubuntu, electron, ai
description: Porting the official ComfyUI Desktop app to Ubuntu 26.04, fixing Linux packaging, Electron runtime issues, and existing-install detection.
---

The official ComfyUI Desktop app ships for Windows and macOS.

I wanted it on Ubuntu 26.04.

So I ported it.

Not a web wrapper. Not a fake launcher. Not a shell script around a browser tab.

The actual desktop app.

## The Goal

I had two requirements:

1. build the real app on Linux
2. make it usable on a machine that already runs ComfyUI normally

That second one mattered more than people think.

If a Linux user already has:

- a real `~/ComfyUI` install
- a running server on `127.0.0.1:8188`

then the desktop app should not pretend the machine is blank and try to push a fresh install into `~/Documents/ComfyUI`.

That is bad product behavior, even if the package technically launches.

## The Starting Point

The upstream repo is here:

https://github.com/Comfy-Org/desktop

At first glance it looks cross-platform enough:

- Electron app
- bundled frontend assets
- `uv` bootstrap
- asset build pipeline
- electron-builder config with Linux bits already half-present

But “half-present” is the dangerous state.

That usually means:

- some paths exist
- some assets exist
- some packaging config exists
- and the app still does not actually ship on Linux

Which is exactly what was going on here.

## The Real Problems

There were a few distinct failures.

### 1. Linux was gated out of asset/bootstrap flow

The app would build fine for supported platforms, but Linux was not being carried all the way through asset setup and build verification.

That meant some Linux artifacts were never properly prepared even though the source tree looked close.

### 2. Packaged builds were missing `desktop-ui`

This was the first hard packaging bug.

The app launched, then died with a file-not-found error because the packaged build did not contain the desktop onboarding UI bundle.

That is the kind of bug that makes a port look dead before you even get to runtime behavior.

### 3. AppImage hit Electron sandbox issues

This is the standard Linux Electron tax.

The AppImage mounted fine, but Chromium tripped over the sandbox helper and aborted before the app could become useful.

I did not waste time pretending AppImage was the right hill to die on.

For Ubuntu, `.deb` was the better target.

### 4. Linux runtime assumptions were wrong

The app still had platform checks and install behavior that were effectively “Windows/macOS first, Linux later.”

A Linux port cannot just compile. It has to stop acting like Linux is unsupported at runtime.

### 5. Existing Comfy installs were ignored

This was the most important UX issue.

On my machine, ComfyUI was already installed and already running in the browser.

The desktop app still tried to start from scratch and push a managed install path into `Documents`.

That is technically launchable, but functionally wrong.

So I fixed it.

## What I Changed

I split the work into build fixes, runtime fixes, and “make it behave like a sane Linux app” fixes.

### Build and packaging fixes

- enabled Linux asset bootstrap
- enabled Linux build verification
- packaged the missing `desktop-ui` bundle
- switched the Linux target to `.deb`

### Runtime fixes

- added Linux config/reset path handling
- made Linux hardware validation non-fatal
- added fallback behavior for missing compiled requirements
- added Linux-safe Electron startup defaults

### Existing-install behavior

This was the good part.

I changed the app so it now:

- prefers an existing `~/ComfyUI` directory as the default base path
- detects a live local Comfy server on `127.0.0.1:8188`
- persists external-server mode into desktop config
- skips the desktop-managed venv validation path when attaching to that external server
- opens the frontend against the already-running instance instead of forcing first-run install flow

That last bit is what makes the port feel real instead of merely “bootable.”

## Why `.deb` Won

I know someone will ask why I did not just ship AppImage.

Because Ubuntu users need something that works, not something ideologically pure.

The `.deb` route gave me:

- stable installation under `/opt/ComfyUI`
- normal desktop integration
- no AppImage sandbox headache
- a clean launcher path

On Linux, packaging format is not theology. It is operational reality.

## What the App Does Now

On an Ubuntu 26.04 machine with an existing Comfy setup:

- launch the desktop app
- it detects `~/ComfyUI`
- it sees the live server on `127.0.0.1:8188`
- it stores that as the desktop install state
- it attaches instead of onboarding

That is the behavior I wanted from the start.

## Commands I Actually Used

Install dependencies:

```bash
corepack enable
corepack yarn install
```

Bootstrap app assets:

```bash
corepack yarn make:assets
```

Verify:

```bash
corepack yarn typecheck
corepack yarn vitest run
```

Build the package:

```bash
corepack yarn make
```

Install the package:

```bash
sudo apt install ./dist/ComfyUI-0.8.30-amd64.deb
```

## The Nice Surprise

The upstream repo was not hopeless.

This was not one of those situations where “Linux support” secretly meant rewriting the app from scratch.

Most of the work was:

- finishing incomplete Linux paths
- fixing packaging omissions
- picking the right distribution target
- making startup logic respect an existing local Comfy workflow

That is still real work, but it is not fantasy.

## The Useful Lesson

A port is not finished when the window appears.

A port is finished when the app behaves correctly on the target machine.

For this one, the difference was:

- not just “ComfyUI Desktop launches on Ubuntu”
- but “ComfyUI Desktop understands an Ubuntu user who already runs ComfyUI”

That is the difference between a demo and a port.

## Upstream and Source

- Upstream desktop app: https://github.com/Comfy-Org/desktop
- ComfyUI core: https://github.com/comfyanonymous/ComfyUI

If I publish the fork cleanly, I will add the repo link here.
