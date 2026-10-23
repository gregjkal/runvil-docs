# Install

This chapter installs Workshop, the desktop app for writing prompts, running them, and reviewing every run, and explains how it stays up to date.

## Download the installer

Download the installer for your machine from the [project README](https://github.com/cppalliance/runvil#readme):

- Windows x64: `Runvil-setup.exe`.
- macOS: `Runvil-arm64.dmg` on Apple silicon or `Runvil-x64.dmg` on Intel. Open the disk image and run `Runvil Installer`.
- Linux: `Runvil-x64.run` on Ubuntu 22.04, Debian 12, or later, or `Runvil-arm64.run` on Ubuntu 24.04, Debian 13, or later. Make it executable and run it from your desktop session.

The [latest release](https://github.com/cppalliance/runvil/releases/latest) holds the same installers under versioned names, with a `SHA256SUMS` file to check them against.

The installers are not code-signed yet. On Windows, SmartScreen may stop the setup: choose More info, then Run anyway. On macOS the first open is refused: open System Settings, then Privacy & Security, choose Open Anyway, and open the installer again.

## Choose components

The installer offers three components, all checked by default:

- Workshop: the desktop app.
- Gateway: the gateway, the one process that holds model credentials and serves models to Workshop. It runs in the system tray, and its Settings page is where you add providers and keys. The Gateway set of this guide covers it.
- Speech to Text: the speech runtime and two speech models for dictation, downloaded during the install, about 0.6 GB, more on Windows and Linux x64 with an NVIDIA GPU. It needs Gateway.

Keep Workshop and Gateway together unless a gateway already runs on another machine. Workshop installed without Gateway attaches to a gateway already running for you, or else uses the one `~/.runvil/workshop.toml` names; when neither exists, Workshop stops at start with an error dialog that says what to set:

````
[gateway]
base_url = "http://gateway.example:8081"
api_key = "the gateway's bearer key"
````

## Where it goes

- Windows: `%LOCALAPPDATA%\Runvil`, with a Start menu entry for Workshop.
- macOS: `/Applications/Runvil`, holding `Runvil.app` and `Runvil Gateway.app`. The installer never asks for an administrator password; if you cannot write `/Applications`, choose a folder you can, such as `~/Applications/Runvil`, and keep both apps in that one folder, since Workshop finds the gateway beside it.
- Linux: `~/Runvil`, holding `Runvil.AppImage` and `runvil-gateway`, with "Runvil" and "Runvil Gateway Settings" entries in your applications menu. A minimal or server system may lack libraries every desktop has, such as EGL, GLES, OpenGL, HarfBuzz, FriBidi, fontconfig, the Wayland client library, and the X11 (xcb) libraries the installer itself uses.

Your settings, the gateway's config and keys, logs, and downloaded models live in the `.runvil` directory of your user profile, not in the install directory.

## Change or remove components

On Windows, run the setup again to add components; its components page starts from your previous choice. To remove a component, uninstall Runvil from Settings, Apps, then run the setup again with only the components you want. Uninstalling keeps the `.runvil` directory unless you tick the uninstaller's Delete app data check box.

On macOS and Linux, the installer refuses a folder that already holds an installation. To change components, run `maintenancetool` (`maintenancetool.app` on macOS) in the install directory to uninstall, then run the installer again with the components you want. Uninstalling removes the install directory and its menu entries, so your `.runvil` directory and its downloaded models stay.

To add speech later, add `[[stt_model]]` entries to the gateway's config, list them in the selected profile's `models`, and run the gateway's `init` command; the Gateway set's install and speech chapters cover it. Checking Speech to Text in a later install does not add them, since the installer keeps your existing config.

## Updates

Workshop checks for an update a few seconds after it starts, and whenever you choose Check for Updates in the About dialog, under Help. When a new version exists, a banner offers it: choose Download Update, and Workshop downloads it, installs it, and restarts. On macOS and Linux it replaces its own app, and on Windows it runs the new setup, which updates every installed component.

On macOS and Linux, after each check, whether or not Workshop itself has an update, Workshop asks the local gateway it uses to update too, and the gateway installs any newer version at once. Workshop shows a notice once the gateway has installed the new version and is restarting, waits while it restarts, and then reconnects on its own.

A gateway installed without Workshop updates itself on request from its tray menu; the Gateway set's install chapter covers it.
