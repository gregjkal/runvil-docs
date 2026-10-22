# Install

This chapter installs Workshop, the desktop app for writing prompts, running them, and reviewing every run, and explains how it stays up to date.

## Download the installer

Download the installer for your machine from the [project README](https://github.com/cppalliance/runvil#readme):

- Windows x64: `Runvil-setup.exe`.
- macOS: `Runvil-arm64.dmg` on Apple silicon or `Runvil-x64.dmg` on Intel. Open the disk image and run `Runvil Installer`.
- Linux: `Runvil-x64.run` on Ubuntu 22.04 or later, or `Runvil-arm64.run` on Ubuntu 24.04, Debian 13, or later. Make it executable and run it from your desktop session.

The [latest release](https://github.com/cppalliance/runvil/releases/latest) holds the same installers under versioned names, with a `SHA256SUMS` file to check them against.

The installers are not code-signed yet. On Windows, SmartScreen may stop the setup: choose More info, then Run anyway. On macOS the first open is refused: open System Settings, then Privacy & Security, choose Open Anyway, and open the installer again.

## Choose components

The installer offers three components, all checked by default:

- Workshop: the desktop app.
- Gateway: the gateway, the one process that holds model credentials and serves models to Workshop. It runs in the system tray, and its Settings page is where you add providers and keys. The Gateway set of this guide covers it.
- Speech to Text: the speech runtime and two speech models for dictation, downloaded during the install, about 0.6 GB, more with an NVIDIA GPU. It needs Gateway.

Keep Workshop and Gateway together unless a gateway already runs on another machine. Workshop installed without Gateway uses the gateway that `~/.runvil/workshop.toml` names; until that file names one, Workshop stops at start with an error dialog that says what to set:

````
[gateway]
base_url = "http://gateway.example:8081"
api_key = "the gateway's bearer key"
````

## Where it goes

- Windows: `%LOCALAPPDATA%\Runvil`, with a Start menu entry for Workshop.
- macOS: `/Applications/Runvil`, holding `Runvil.app` and `Runvil Gateway.app`. The installer never asks for an administrator password; if you cannot write `/Applications`, choose a folder you can, such as `~/Applications/Runvil`, and keep both apps in that one folder, since Workshop finds the gateway beside it.
- Linux: `~/Runvil`, holding `Runvil.AppImage` and `runvil-gateway`, with "Runvil" and "Runvil Gateway Settings" entries in your applications menu. A minimal or server system may lack libraries every desktop has, such as EGL, GLES, OpenGL, HarfBuzz, FriBidi, fontconfig, and the Wayland client library.

Your settings, the gateway's config and keys, logs, and downloaded models live in the `.runvil` directory of your user profile, not in the install directory.

## Change or remove components

On Windows, run the setup again; its components page starts from your previous choice. Uninstall from Settings, Apps, where "Delete app data" also removes the `.runvil` directory.

On macOS and Linux, the installer refuses a folder that already holds an installation. To change components, run `maintenancetool` in the install directory to uninstall, then run the installer again with the components you want. Uninstalling removes only the install directory, so your `.runvil` directory and its downloaded models stay.

To add speech later without reinstalling, declare `[[stt_model]]` entries in the gateway's selected profile and run `runvil-gateway init` from the install directory; the Gateway set's speech chapter covers it.

## Updates

Workshop checks for an update a few seconds after it starts, and whenever you choose Check for Updates in the About dialog, under Help. It installs a new version for you: on macOS and Linux it replaces its own app, and on Windows it runs the new setup, which updates every installed component.

After its own check, Workshop asks the local gateway it uses to update itself too, and shows a notice when the gateway starts updating. Workshop stays connected while the gateway restarts into the new version.

A gateway installed without Workshop updates itself on request from its tray menu; the Gateway set's install chapter covers it.
