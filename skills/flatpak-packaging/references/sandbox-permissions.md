# Sandbox Permissions

Related public docs: [Sandbox Permissions](https://docs.flatpak.org/en/latest/sandbox-permissions.html), [Portal support](https://docs.flatpak.org/en/latest/portals.html), [Desktop Integration](https://docs.flatpak.org/en/latest/desktop-integration.html), [Debugging](https://docs.flatpak.org/en/latest/debugging.html).

## Default sandbox

By default, a Flatpak app cannot access host files except its runtime, its app files, `~/.var/app/$FLATPAK_ID`, and `$XDG_RUNTIME_DIR/app/$FLATPAK_ID`; only the latter two are writable. It also lacks network, device nodes, host processes, unrestricted syscalls, unfiltered D-Bus, X11, system D-Bus, and PulseAudio unless permissions are added.

Configure app permissions in manifest `finish-args`. These arguments are static; do not expect variable expansion or substitution.

## Prefer portals

Use portals before static permissions when the app needs user-mediated access to files, URIs, printing, screenshots, notifications, network status, inhibit, and related desktop services. GTK and Qt/KDE can use many portals transparently. Apps without toolkit support can call xdg-desktop-portal directly.

Portal use often avoids broad grants such as `--filesystem=home` or `--talk-name=org.freedesktop.Notifications`.

## Standard GUI permissions

Common GUI set for Wayland-capable apps:

```yaml
finish-args:
  - --share=ipc
  - --socket=fallback-x11
  - --socket=wayland
  - --device=dri
```

X11-only apps:

```yaml
finish-args:
  - --share=ipc
  - --socket=x11
  - --device=dri
```

Add only when required:

- `--share=network`: network access.
- `--socket=pulseaudio`: audio playback/input, MIDI, ALSA sound devices.
- `--socket=cups`: CUPS printing access.
- `--allow=bluetooth`: Bluetooth sockets.
- `--socket=ssh-auth`: SSH agent, usually for Git/SSH clients.
- `--socket=gpg-agent`: GPG agent, usually for mail or GPG frontends.

`--socket=inherit-wayland-socket` is sensitive and should be rare.

## D-Bus

Session bus access is filtered by default. The app can own and talk within its own `$FLATPAK_ID` namespace, `org.mpris.MediaPlayer2.$FLATPAK_ID`, `org.freedesktop.DBus`, and portal APIs.

Avoid:

- `--socket=session-bus`
- `--socket=system-bus`

They disable filtering and are security risks except for development/system tools.

Prefer narrow permissions:

```yaml
finish-args:
  - --talk-name=org.example.Service
  - --system-talk-name=org.example.SystemService
  - --own-name=org.example.App.Helper
```

Find needed names with:

```bash
flatpak run --log-session-bus org.example.App
flatpak run --log-session-bus org.example.App | grep '(required 1)'
```

## Filesystem

Prefer XDG app data paths and portals. If static access is necessary:

- Add `:ro` for read-only access.
- Use specific XDG directories such as `xdg-documents`, `xdg-download`, `xdg-music`, or `xdg-pictures`.
- Use subpaths, for example `--filesystem=xdg-documents/MyApp`.
- Avoid `home` and `host` unless the application is a file manager, editor, IDE, backup tool, or similar app where broad access is intrinsic.

Examples:

```yaml
finish-args:
  - --filesystem=xdg-documents:ro
  - --filesystem=xdg-download/MyApp:create
```

Special filesystem grants include:

- `home`: home directory except `~/.var/app`.
- `host`: most top-level host paths except reserved paths.
- `host-os`: host OS paths mounted under `/run/host`.
- `host-etc`: host `/etc` mounted under `/run/host/etc`.
- `host-root`: complete host OS at `/run/host/root` on Flatpak 1.17.0+.
- `xdg-config`, `xdg-cache`, `xdg-data`, `xdg-run/path`: XDG locations.

Reserved paths do not work with `--filesystem`: `/app`, `/bin`, `/dev`, `/etc`, `/lib`, `/lib32`, `/lib64`, `/proc`, `/run/flatpak`, `/run/host`, `/sbin`, `/usr`, and most of `/run`.

Use `--persist=DIR` when an app hardcodes a home-relative directory and you want persistence without full home access:

```yaml
finish-args:
  - --persist=.myapp
```

This maps sandbox `~/.myapp` to `~/.var/app/$FLATPAK_ID/.myapp` on the host.

## Devices

Narrow device permissions:

- `--device=dri`: GPU/GL.
- `--device=kvm`: `/dev/kvm`.
- `--device=shm`: shared memory in `/dev/shm`.
- `--device=input`: input devices, including game controllers, Flatpak 1.15.6+.
- `--device=usb`: raw USB bus, Flatpak 1.15.11+.

Avoid `--device=all` unless no narrower permission is available, for example some webcam or optical drive use cases.

## USB portal

For libusb-style raw USB access on Flatpak 1.15.11+, prefer the USB portal when feasible. Use `--usb=` to enumerate allowed devices and `--nousb=` to hide exceptions. Opening still requires a portal permission request.

Examples:

```yaml
finish-args:
  - --usb=vnd:1234
  - --usb=vnd:1234+dev:3456
  - --usb=cls:06:*
  - --nousb=vnd:1234+dev:9999
```

Use `--usb-list` or `--usb-list-file` for long lists.

## DConf and GVfs

Modern GLib and xdg-desktop-portal usually avoid direct DConf access. For migration from older apps:

```yaml
finish-args:
  - --metadata=X-DConf=migrate-path=/org/example/foo/
```

Direct DConf access for old runtimes:

```yaml
finish-args:
  - --filesystem=xdg-run/dconf
  - --filesystem=~/.config/dconf:ro
  - --talk-name=ca.desrt.dconf
  - --env=DCONF_USER_CONFIG_DIR=.config/dconf
```

For GVfs:

- GTK/GNOME apps typically need `--talk-name=org.gtk.vfs.*` plus `--filesystem=xdg-run/gvfsd`.
- Legacy/non-GIO apps typically need `--filesystem=xdg-run/gvfs`.
- Do not use `--talk-name=org.gtk.vfs`; use the wildcard form.

## External drives

Common mounts:

```yaml
finish-args:
  - --filesystem=/media
  - --filesystem=/run/media
  - --filesystem=/mnt
```

Avoid subpaths unless they are predictable across systems.

## Conditional permissions

Flatpak 1.17.0+ supports conditional permissions:

```yaml
finish-args:
  - --socket=x11
  - --socket-if=x11:!has-wayland
  - --device=all
  - --device-if=all:!has-input-device
  - --device=input
```

Available conditional forms: `--socket-if=`, `--device-if=`, `--share-if=`, `--allow-if=`. Conditions include `true`, `false`, `has-input-device`, and `has-wayland`; prefix `!` to negate.

Use `--nosocket=NAME`, `--unshare=NAME`, and similar denies to remove inherited grants from runtime metadata or overrides.
