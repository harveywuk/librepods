# librepods (patched fork)

A fork of [librepods](https://github.com/kavishdevar/librepods) by **Kavish
Devar**, who reverse-engineered Apple's AAP protocol over L2CAP and the BLE
advertisement path that carries battery, in-ear and case-lid state. That is the
hard part, and it is his.

This repository carries the Linux daemon plus the patches the
[AirPods Noctalia plugin](https://github.com/harveywuk/airpods) needs. Upstream
does not have them, so nothing packaged will do.

## What the patches add

- A **published state file** — one line of JSON at
  `$XDG_STATE_HOME/librepods/status.json`, written on change and removed on quit.
- A **`status` verb** and the **`ca:`**, **`onebud:`**, **`adaptive:N`** and
  **`ear:`** control verbs (upstream has only the noise verbs).
- **AirPods Pro 3 support** and a current model map, including a
  `supports_noise_off` flag (the Pro 3 has no Off mode).
- **Case-lid state** from BLE advertisements.
- The **control socket moved off `/tmp`** to `$XDG_RUNTIME_DIR/librepods.sock`
  (mode 0700).
- A **systemd user unit** bound to `graphical-session.target`, and a
  **`--headless` mode** that builds no tray or QML engine.
- Notifications through the host desktop rather than a Qt tray toast.

See [UPSTREAM.md](UPSTREAM.md) for the exact fork point and the full change log.

## Build

```bash
cmake -B build -G Ninja -DBUILD_TESTING=OFF
cmake --build build
cmake --install build --prefix ~/.local
systemctl --user daemon-reload
systemctl --user enable librepods.service
systemctl --user restart librepods.service
```

Dependencies (Arch): `cmake`, `ninja`, `pkgconf`, `qt6-connectivity`,
`qt6-tools`, `qt6-declarative`, `libpulse`, `openssl`.

`~/.local` is the prefix the unit expects (`%h/.local/bin/librepods`).

### `--no-ble-scan`

While the AirPods are disconnected the daemon runs BLE discovery continuously
to read case-lid and case-battery advertisements. Every nearby BLE device then
wakes `bluetoothd`, D-Bus, the daemon and any audio service watching BlueZ
(WirePlumber took ~17% of a core on a laptop). `--no-ble-scan` turns the scan
off; connection, battery, listening modes and ear detection keep working while
connected. Add it to `ExecStart` in the unit if you don't need the
disconnected-state case info.

## License

GPL-3.0, inherited from upstream. The original librepods is by **Kavish Devar**;
the patches on top of it are by **GM (thisisgm)**, from
[thisisgm/omarchy-pods](https://github.com/thisisgm/omarchy-pods).
