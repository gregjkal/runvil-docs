# Install and Run

This chapter teaches you to get the gateway running on your machine: how to install it, how to start it with a config file and a profile, and how to confirm it is healthy. You do these things every time you bring the gateway up, so they are worth learning well.

## Install the gateway

The gateway is a single program named `runvil-gateway`. It serves an OpenAI-shaped inference API. On a desktop machine, install it with the Runvil installer for your platform, linked from the [project README](https://github.com/cppalliance/runvil#readme): `Runvil-setup.exe` on Windows, a `.dmg` on macOS, and a `.run` file on Linux.

The installer offers three components. Gateway installs the gateway. Workshop installs the desktop app, which uses the gateway beside it. Speech to Text needs Gateway and downloads the speech runtime and two speech models during the install, about 0.6 GB, so speech works offline from the first start. Check only Gateway for a gateway without Workshop, or Gateway and Speech to Text.

Each platform installs in its own place:

- Windows: `%LOCALAPPDATA%\Runvil\runvil-gateway.exe`. A Gateway-only install's last page offers to open the Settings page in your browser. The Start menu entry belongs to Workshop, so start the gateway later by running `runvil-gateway.exe`, or turn on Launch at Login from its tray menu.
- macOS: `Runvil Gateway.app` in `/Applications/Runvil`. The installer never asks for an administrator password; if you cannot write `/Applications`, choose a folder you can, such as `~/Applications/Runvil`.
- Linux: `runvil-gateway` in `~/Runvil`, with a "Runvil Gateway Settings" entry in your applications menu. The x64 installer runs on Ubuntu 22.04 or later and the ARM64 installer on Ubuntu 24.04, Debian 13, or later, from a desktop session. A minimal or server system may lack libraries every desktop has, such as EGL, GLES, OpenGL, HarfBuzz, FriBidi, fontconfig, the Wayland client library, and the X11 (xcb) libraries the installer itself uses.

The installers are not code-signed yet. On Windows, SmartScreen may stop the setup: choose More info, then Run anyway. On macOS the first open is refused: open System Settings, then Privacy & Security, choose Open Anyway, and open the installer again. Until the apps are signed, Launch at Login does not work on macOS.

The installers do not put the gateway on your `PATH`, so run it by its full path. In the default install folder that is:

- Windows, in PowerShell: `& "$env:LOCALAPPDATA\Runvil\runvil-gateway.exe"`
- macOS: `"/Applications/Runvil/Runvil Gateway.app/Contents/MacOS/runvil-gateway"`
- Linux: `~/Runvil/runvil-gateway`

The commands in this guide write that path as `runvil-gateway`. Confirm the install by printing the version:

````
runvil-gateway --version
````

Installing the gateway runs `runvil-gateway init`, which writes the default config described below when none exists and, with Speech to Text checked, downloads the speech runtime and the speech models the selected profile lists. When that step fails, for example without a network, the Windows setup keeps the gateway installed and says so; on macOS and Linux, choose Ignore to keep the installation, since Cancel removes it. Then run `runvil-gateway init` yourself to retry. `init --no-stt` writes the config without downloading anything; `init` with speech refuses a config whose selected profile lists no `[[stt_model]]`, and names the profile.

Your config, keys, logs, and downloaded models live in the `.runvil` directory of your user profile, not in the install directory, so uninstalling keeps them, unless you tick Delete app data in the Windows uninstaller. On Windows, rerun the setup to change components; its components page starts from your previous choice. On macOS and Linux, the installer refuses a folder that already holds an installation: run `maintenancetool` in the install directory to uninstall, then run the installer again with the components you want.

On a server, build the gateway from a source checkout instead. Install the Settings page's dependencies, then build:

````
npm ci --prefix crates/gateway/config-ui/ui
cargo build --locked --release -p gateway
````

The program lands in `target/release/runvil-gateway`.

## Start the gateway

Start the gateway by naming a config file and a profile:

````
runvil-gateway --config gateway.toml --profile main
````

The `--config` flag gives the path to the config file. The `--profile` flag names the profile to activate. The gateway always starts from one config file and one active profile.

Startup is bind-first. The gateway opens its listener and answers health, status, progress, configuration, and every ready route immediately, then provisions models afterward as one queued boot command. Downloads, local model spawns, and the speech engine load all run inside that command while the gateway is already serving, and you can watch it on the status and progress endpoints. A configured model that is still loading answers 503 with the code `model_loading` until its provisioning finishes.

You can supply both values through environment variables instead of command-line arguments. The config path comes from `--config` or from `RUNVIL_GATEWAY_CONFIG`; the flag wins when both are set. The profile comes from `--profile`, then `RUNVIL_PROFILE`, then the sibling state file the gateway keeps beside the config.

You can also start the gateway with no config file at all. When no `gateway.toml` exists beside the executable, in the working directory, or in the user profile's `.runvil` directory, the first run writes a default config there - loopback-only on an OS-assigned port, with a fresh random bearer key and `trust_loopback = true` so callers on the same machine need no key - and boots from it. The generated file notes the caveat beside that line: on a shared machine any other OS account can then use the gateway, and `trust_loopback = false` requires the key from everyone. The generated config selects a profile named `default`, so a bare first boot needs no flags.

## The system tray

On a desktop system the gateway's face is the system tray. The icon shows the gateway's state, and its menu lists a status line, a Workshop item that launches the Workshop application when the installer laid it beside the gateway, a Settings item that opens the configuration UI in your browser, an update item, a Launch at Login toggle, and Quit. A gateway started at login never opens a browser or a window.

For servers and CI, `--no-tray` keeps the plain headless loop. In a tray-less environment, `--print-url` prints the Settings URL to stdout once the gateway is bound. `--browser` opens the Settings page in your default browser once bound; the installer uses it on a Gateway-only install's first run. Launching `runvil-gateway` while one is already running never starts a second copy: it opens the running gateway's Settings page instead.

After every successful bind the gateway writes a gateway discovery file (`gateway.json` in the run directory under the state directory) that records its port, bearer key, and process id. Runvil components read that file to attach to the running gateway instead of starting a second one, and a clean shutdown removes it.

## Update the gateway

An installed gateway updates itself from the same signed release channel Workshop uses, and only ever to a newer version whose signature checks out against the release key built into it.

A gateway in the tray checks for an update when it starts and then every day, an hour later when a check fails. When a newer version exists, the status line and the update item read "Update available" with the version. The tray never installs on its own, because a restart drops loaded models and requests in flight: choose the update item to install it. The item reads "Check for Updates" before a check finds anything and "Up to date" with the version after one finds nothing; choosing either checks again and installs whatever it finds. While an update downloads, the status line shows its progress, and a failed check or update shows "Update failed" with the reason.

Workshop asks as well, and the gateway installs at once when it does: after each of Workshop's update checks, including the automatic one a few seconds after Workshop starts, Workshop asks the local gateway it uses to update, and shows "Runvil Gateway is updating to" the new version when one installs. On Windows, Workshop's update reinstalls every component, so the gateway's item there reads "Updates come from Runvil Workshop" and its own check never runs.

On macOS and Linux the gateway replaces its own app or program, starts the new version, and exits once the new one is serving; if the new one never comes up, it puts the old one back. Workshop waits through the switch. On Windows without Workshop, the gateway downloads the new setup and runs it, and the setup stops and restarts the gateway.

A gateway run with `--no-tray`, including the systemd service below, never checks for updates on its own, though it still updates when a local Workshop asks it to. Update a server by rebuilding it from source.

## Check that it is healthy

Once the gateway is serving, probe its health endpoint:

````
curl http://127.0.0.1:8081/health
````

GET /health needs no credentials. It always answers 200 while the gateway is serving.

Every /v1 route is authenticated with the shared bearer key from the config file. The address in this request is the `bind` value from the `[server]` section of the config file; `127.0.0.1:8081` is an example bind. A request with a wrong token is rejected with status 401 and error code `unauthorized`, from any peer:

````
curl -H "Authorization: Bearer wrong-token" http://127.0.0.1:8081/v1/models
````

From the gateway's own machine you can leave the key out entirely. With the default `trust_loopback = true`, a loopback request that presents no credential is admitted:

````
curl http://127.0.0.1:8081/v1/models
````

This convenience has one cost: on a shared machine, any other OS account can use the gateway the same way, including reading upstream API keys from the admin config surface. Set `trust_loopback = false` in `[server]` to require the key from every caller. The configuration chapter covers the rule in full.

## Choose what to build

Build-time feature flags decide which capabilities exist in the binary. The flags `local`, `web-search`, `stt`, and `config-ui` are on by default. A headless build without `local` refuses any configuration that declares local models; the refusal happens at startup.

## Run it as a service on Linux

A source checkout holds a sample systemd unit, `crates/gateway/app/packaging/gateway.service`. The unit runs the gateway as a service with a fixed config path and profile, and restarts it automatically on failure:

````
ExecStart=/usr/local/bin/runvil-gateway --config /etc/runvil/gateway.toml --profile main --no-tray
Restart=on-failure
RestartSec=5
````

The gateway holds vendor credentials, so run it as a dedicated unprivileged user. The sample unit does this with `DynamicUser=yes` and keeps state in a systemd-managed state directory (`StateDirectory=runvil`).

## Watch the logs

A serving gateway logs to `gateway.log` in the `logs` directory under the state directory (`~/.runvil/logs` on a default install) and mirrors the same stream to stdout. Startup rotates the previous run's log aside - `gateway.log` becomes `gateway.log.1` - and keeps five previous runs, deleting the oldest. Every record crosses a redaction pass before it reaches disk: bearer tokens, authorization and cookie header values, and `api_key` assignments are masked. The log location is never configurable, so a config failure still has somewhere to report itself. On macOS and Linux, the new version's start rotates the log during an update, so the outgoing gateway's last lines about the update are in `gateway.log.1`.

Control log verbosity through the standard `RUST_LOG` environment filter. The speech library logs at warn level by default, so it stays quiet unless you ask for more.

Startup failures appear on stderr with the full cause chain: one `error:` line followed by one `caused by:` line per cause, and the same chain lands in the log file. Once the gateway is serving, the log shows the bound address. If you configured port 0, the log reports the real bound port.

## Inspect a failed run

When a gateway run fails before it can serve, `runvil-gateway diagnostics` finds the evidence without any config knowledge. It prints a read-only JSON report: the state directory, the resolved config path and whether it exists, the current and retained log paths and which exist, the gateway discovery file, whether a gateway is running, and the version. It never serves, rotates a log, parses a config, or mutates the state directory, and it never prints secrets - no bearer key, environment value, config content, or log content. The generated config points at it in a comment.

## Stop the gateway

From the tray, choose Quit. From a script or another Runvil component, send an authenticated POST to the `/shutdown` route with the bearer key; it answers 202 and then the server goes down. Under `--no-tray`, Ctrl-C stops the gateway cleanly. Every path drains in-flight requests before the exit.

