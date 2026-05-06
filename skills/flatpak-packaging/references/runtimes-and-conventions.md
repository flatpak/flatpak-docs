# Runtimes and Conventions

Related public docs: [Available Runtimes](https://docs.flatpak.org/en/latest/available-runtimes.html), [Basic concepts](https://docs.flatpak.org/en/latest/basic-concepts.html), [Dependencies](https://docs.flatpak.org/en/latest/dependencies.html), [Requirements & Conventions](https://docs.flatpak.org/en/latest/conventions.html), [Desktop Integration](https://docs.flatpak.org/en/latest/desktop-integration.html).

## Runtime choice

Every app needs a runtime and matching SDK. The runtime is the app's stable cross-distribution base; the SDK is the build environment with compilers, headers, tools, and debuggers.

Main runtimes:

- Freedesktop: `org.freedesktop.Platform` and `org.freedesktop.Sdk`. Generic base for any app, graphics/toolchain stack, parent of GNOME and KDE runtimes, strict ABI/API stability within a major branch.
- GNOME: `org.gnome.Platform` and `org.gnome.Sdk`. Use for GNOME/GTK apps when the GNOME platform provides needed libraries.
- KDE: `org.kde.Platform` and `org.kde.Sdk`. Use for Qt/KDE apps; includes Qt and KDE Frameworks.
- elementary: `io.elementary.Platform` and `io.elementary.Sdk`. Use for elementary AppCenter targets.

Prefer an existing runtime over creating a custom runtime. Bundle dependencies missing from the chosen runtime, but keep bundled modules minimal.

Check runtime contents:

```bash
flatpak run --command=cat org.gnome.Platform//47 /usr/manifest.json | jq -r '."modules"|.[]|."name"' | sed -E 's#.*/(.*)\.bst#\1#' | sort -u
flatpak run --command=pkg-config org.freedesktop.Sdk//24.08 --modversion appstream
flatpak run --command=ldconfig org.freedesktop.Platform//24.08 -p | awk '/\.so/ {print $1}'
```

## Application IDs

Use a unique reverse-DNS ID such as `org.example.App`. The domain portion must be lowercase. Avoid:

- IDs ending in `.desktop`.
- Bare hosting namespaces such as `io.github.foo`; prefer `io.github.foo.foo`.
- `com.github.*` or `com.gitlab.*` for projects not controlling those domains.
- Dashes in invalid D-Bus name components; use underscores when needed.
- Capitalized domain portions such as `Org.Example.App`.

The same ID should be used for manifest filename, desktop file, icon name, MetaInfo ID, exported service names, and app namespace where possible.

## MetaInfo/AppStream

Install MetaInfo to:

```text
/app/share/metainfo/org.example.App.metainfo.xml
```

Validate:

```bash
appstreamcli validate --explain org.example.App.metainfo.xml
```

Flatpak Builder composes metadata with AppStream during build. App stores can impose additional requirements.

## Icons

Install icons named with the app ID:

```text
/app/share/icons/hicolor/128x128/apps/org.example.App.png
/app/share/icons/hicolor/scalable/apps/org.example.App.svg
```

Icons must be square. PNG and SVG are accepted. Flatpak exports `$FLATPAK_ID`, `$FLATPAK_ID.foo`, and `$FLATPAK_ID-foo` icon patterns.

## Desktop files

Install desktop file:

```text
/app/share/applications/org.example.App.desktop
```

Minimal shape:

```ini
[Desktop Entry]
Name=Example App
Exec=app-command
Type=Application
Icon=org.example.App
Categories=Utility;
```

Validate:

```bash
desktop-file-validate org.example.App.desktop
```

Flatpak rewrites `Exec` at install time to invoke `flatpak run --command=...`.

## D-Bus service files

Install service files to:

```text
/app/share/dbus-1/services/org.example.App.service
```

The filename before `.service` must match `$FLATPAK_ID` or a subname. The `Name=` key must match exactly.

## Search providers, Krunner, and MIME

GNOME Shell search provider:

```text
/app/share/gnome-shell/search-providers/org.example.App-search-provider.ini
```

Krunner D-Bus plugin:

```text
/app/share/krunner/dbusplugins/org.example.App.desktop
```

MIME package:

```text
/app/share/mime/packages/org.example.App-mime.xml
```

Flatpak may disable or rewrite some exported integration files for safety.

## XDG base directories

Inside the sandbox, Flatpak sets:

- `XDG_CONFIG_HOME`: `~/.var/app/<app-id>/config`
- `XDG_DATA_HOME`: `~/.var/app/<app-id>/data`
- `XDG_CACHE_HOME`: `~/.var/app/<app-id>/cache`
- `XDG_STATE_HOME`: `~/.var/app/<app-id>/.local/state` on Flatpak 1.13+

Host values may be exposed as `HOST_XDG_CONFIG_HOME`, `HOST_XDG_DATA_HOME`, `HOST_XDG_CACHE_HOME`, and `HOST_XDG_STATE_HOME`.

Prefer toolkit APIs such as GLib XDG helpers, Qt `QStandardPaths`, or Electron `app.getPath`.

## Desktop integration notes

- Portals are the preferred path for files, URIs, printing, notifications, screenshots, and similar host integration.
- Avoid relying on tray/status icons as the only control surface; some Linux desktops do not show them.
- StatusNotifier may need `--talk-name=org.kde.StatusNotifierWatcher` and, for old implementations, `--own-name=org.kde.*`; current Electron, Chromium, KNotifications, and libappindicator usually avoid ownership permissions.
- Host themes are not directly available from `/usr`. Prefer runtime theme extensions and portal/XSettings support over filesystem access to host theme directories.
- Host icons are exposed under `/run/host/share/icons` and `/run/host/user-share/icons`.
- Host fonts are exposed under `/run/host/fonts`, `/run/host/local-fonts`, and `/run/host/user-fonts`; host fontconfig files are not exposed.
