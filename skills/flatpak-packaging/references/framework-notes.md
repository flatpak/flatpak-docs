# Framework Notes

Related public docs: [Python](https://docs.flatpak.org/en/latest/python.html), [Electron](https://docs.flatpak.org/en/latest/electron.html), [Dotnet](https://docs.flatpak.org/en/latest/dotnet.html), [Qt](https://docs.flatpak.org/en/latest/qt.html).

## Python

If the project uses Meson, CMake, or Autotools, use the normal buildsystem. For Setuptools/pip-style projects, use `buildsystem: simple`:

```yaml
- name: python-dep
  buildsystem: simple
  build-commands:
    - pip3 install --prefix=/app --no-deps .
  sources:
    - type: archive
      url: https://files.pythonhosted.org/packages/source/p/pkg/pkg-1.0.tar.gz
      sha256: <sha256>
```

Use `--prefix=/app`; otherwise pip may try to install under read-only `/usr`.

For multiple Python dependencies, prefer `flatpak-pip-generator` from `flatpak-builder-tools`:

```bash
python3 flatpak-pip-generator requests
python3 flatpak-pip-generator --requirements-file=requirements.txt
```

Include the generated JSON in the manifest:

```yaml
modules:
  - python3-requests.json
  - name: app
    buildsystem: meson
    sources: [...]
```

## Electron

Use Freedesktop runtime/SDK and the Electron BaseApp unless there is a specific reason not to:

```yaml
id: org.example.ElectronApp
runtime: org.freedesktop.Platform
runtime-version: '24.08'
sdk: org.freedesktop.Sdk
base: org.electronjs.Electron2.BaseApp
base-version: '24.08'
sdk-extensions:
  - org.freedesktop.Sdk.Extension.node18
build-options:
  append-path: /usr/lib/sdk/node18/bin
command: run.sh
```

Common Electron permissions:

```yaml
finish-args:
  - --share=ipc
  - --device=dri
  - --socket=x11
  - --socket=pulseaudio
  - --share=network
  - --env=ELECTRON_TRASH=gio
```

Wayland support can be enabled with `--socket=wayland` and `--socket=fallback-x11`, but the docs treat native Electron Wayland as experimental; default to X11/Xwayland unless the app and Electron version are known to be stable on Wayland.

Set the desktop file name in `package.json`:

```json
{
  "desktopName": "org.example.ElectronApp.desktop"
}
```

For binary repacks, use the BaseApp's `patch-desktop-filename` on `resources/app.asar`.

Bundle NPM/Yarn dependencies with the Node generator from `flatpak-builder-tools`. It requires `package-lock.json` or `yarn.lock`:

```bash
npm install --package-lock-only
flatpak-node-generator npm package-lock.json
```

Then include the generated source file:

```yaml
sources:
  - generated-sources.json
```

Typical wrapper:

```yaml
- type: script
  dest-filename: run.sh
  commands:
    - zypak-wrapper /app/main/electron-app "$@"
```

Build offline by setting cache paths and `npm_config_offline: 'true'`. If `electron-builder` tries to fetch AppImage or other Linux targets, prefer a build-time `jq` edit to set target `dir`.

Electron notifications use libnotify; modern libnotify uses the notification portal, so `--talk-name=org.freedesktop.Notifications` is often unnecessary.

## .NET

Use a Linux-compatible desktop framework such as Avalonia UI, Uno Platform, Eto, GTKSharp, or Gir.Core. Use the .NET SDK extension and generator from `flatpak-builder-tools`.

Skeleton:

```yaml
id: org.example.DotnetApp
runtime: org.freedesktop.Platform
runtime-version: '24.08'
sdk: org.freedesktop.Sdk
sdk-extensions:
  - org.freedesktop.Sdk.Extension.dotnet8
build-options:
  prepend-path: /usr/lib/sdk/dotnet8/bin
  append-ld-library-path: /usr/lib/sdk/dotnet8/lib
  prepend-pkg-config-path: /usr/lib/sdk/dotnet8/lib/pkgconfig
command: ExampleApp
finish-args:
  - --device=dri
  - --socket=x11
  - --share=ipc
  - --env=DOTNET_ROOT=/app/lib/dotnet
modules:
  - name: dotnet
    buildsystem: simple
    build-commands:
      - /usr/lib/sdk/dotnet8/bin/install.sh
  - name: app
    buildsystem: simple
    sources:
      - type: git
        url: https://example.org/app.git
      - ./nuget-sources.json
    build-commands:
      - dotnet publish ExampleApp.csproj -c Release --no-self-contained --source ./nuget-sources
      - mkdir -p ${FLATPAK_DEST}/bin
      - cp -r bin/Release/net8.0/publish/* ${FLATPAK_DEST}/bin
```

Generate NuGet sources after downloading build deps:

```bash
flatpak-builder build-dir --user --install-deps-from=flathub --download-only org.example.DotnetApp.yml
python3 flatpak-dotnet-generator.py --dotnet 8 --freedesktop 24.08 nuget-sources.json app/ExampleApp.csproj
```

## Qt

Use KDE runtime/SDK for most Qt/KDE apps:

```yaml
id: org.example.QtApp
runtime: org.kde.Platform
runtime-version: '6.9'
sdk: org.kde.Sdk
command: qt-app
finish-args:
  - --share=ipc
  - --socket=fallback-x11
  - --socket=wayland
  - --device=dri
modules:
  - name: qt-app
    buildsystem: cmake-ninja
    config-opts:
      - -DCMAKE_BUILD_TYPE=RelWithDebInfo
    sources:
      - type: archive
        url: https://example.org/qt-app.tar.gz
        sha256: <sha256>
```

If the KDE runtime is too broad or missing needed pieces, use Freedesktop as a base and bundle the Qt parts explicitly, but prefer KDE runtime for ordinary Qt apps.
