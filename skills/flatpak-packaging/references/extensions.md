# Extensions

Related public docs: [Dependencies](https://docs.flatpak.org/en/latest/dependencies.html), [Extensions](https://docs.flatpak.org/en/latest/extension.html), [Available Runtimes](https://docs.flatpak.org/en/latest/available-runtimes.html), [Requirements & Conventions](https://docs.flatpak.org/en/latest/conventions.html).

## When to use extensions

Extensions let runtimes or apps define mount points for optional content. Use them for plugins, add-ons, themes, debug symbols, locales, sources, graphics drivers, SDK toolchains, or optional app functionality.

Flatpak Builder automatically creates `.Debug` and `.Locale` extensions unless disabled. `.Sources` is produced when using `--bundle-sources`; do not define these manually for ordinary apps.

## Define an app extension point

Use `add-extensions` in the app manifest. Extension IDs must start with the extension point prefix.

```yaml
id: org.example.App
runtime: org.gnome.Platform
runtime-version: '48'
sdk: org.gnome.Sdk
command: app
add-extensions:
  org.example.App.Plugin:
    version: '1.0'
    directory: plugins
    add-ld-path: lib
    merge-dirs: plug-ins;help
    subdirectories: true
    no-autodownload: true
    autodelete: true
cleanup-commands:
  - mkdir -p ${FLATPAK_DEST}/plugins
```

Important properties:

- `version`: branch used for matching extension packages; use `versions` for multiple accepted branches.
- `directory`: path relative to `/app` for app extension points or `/usr` for runtime extension points.
- `subdirectories: true`: mount multiple extensions under the same directory.
- `add-ld-path`: add extension library path.
- `merge-dirs`: merge named subdirectories across extensions.
- `no-autodownload: true`: avoid installing every extension automatically.
- `autodelete: true`: remove extensions when the parent app is removed.

Ensure the target directory exists in the final app, often with `cleanup-commands` or `post-install`.

There is no extension property that extends `PATH`; set wrappers or environment/build options explicitly.

## Load existing runtime or SDK extensions

Some extensions are provided by runtimes, such as ffmpeg, GL, i386 compatibility, TeXLive, Java, Go, Node, or .NET SDK extensions. Match extension branch to the base runtime branch.

Find base branch for derived runtimes:

```bash
flatpak remote-info flathub -m org.kde.Platform//5.15-24.08 | grep -A 5 -F '[Extension org.freedesktop.Platform.GL]'
flatpak remote-info flathub -m org.gnome.Sdk//48 | grep -A 5 -F '[Extension org.freedesktop.Sdk.Extension]'
```

Load an existing runtime extension at app runtime:

```yaml
add-extensions:
  org.freedesktop.Platform.ffmpeg-full:
    version: '24.08'
    directory: lib/ffmpeg
    add-ld-path: .
cleanup-commands:
  - mkdir -p ${FLATPAK_DEST}/lib/ffmpeg
```

Use `inherit-extensions` to inherit extension points or extensions from a parent runtime or base:

```yaml
inherit-extensions:
  - org.freedesktop.Platform.GL32
  - org.freedesktop.Platform.ffmpeg-full
```

Use `sdk-extensions` for build-time SDK extensions:

```yaml
sdk-extensions:
  - org.freedesktop.Sdk.Extension.node18
build-options:
  append-path: /usr/lib/sdk/node18/bin
```

Use `add-build-extensions` when a build dependency is itself available as an extension point in the runtime.

## Build an extension manifest

For a separately shipped plugin/add-on:

```yaml
id: org.example.App.Plugin.Foo
branch: '1.0'
runtime: org.example.App
runtime-version: stable
sdk: org.gnome.Sdk//48
build-extension: true
separate-locales: false
build-options:
  prefix: /app/plugins/foo
  prepend-path: /app/plugins/foo/bin
  prepend-pkg-config-path: /app/plugins/foo/lib/pkgconfig
  prepend-ld-library-path: /app/plugins/foo/lib
modules:
  - name: foo
    buildsystem: simple
    build-commands:
      - install -Dm644 org.example.App.Plugin.Foo.metainfo.xml -t ${FLATPAK_DEST}/share/metainfo
      - <build and install commands>
    sources:
      - type: archive
        url: https://example.org/foo.tar.gz
        sha256: <sha256>
```

Rules:

- `id` must use the extension point prefix.
- `branch` must match the extension point `version`.
- `runtime` is the app/runtime that defines the extension point.
- `runtime-version` is the parent branch, often `stable` or `beta` for apps.
- `sdk` should match the SDK used to build the parent runtime/app.
- Use `build-extension: true`.
- Provide MetaInfo with component type `addon` for store discoverability.

## Bundled extensions

Use `bundle: true` to export an extension from the app manifest itself:

```yaml
add-extensions:
  org.example.App.Codecs:
    directory: extensions/codecs
    subdirectories: true
    no-autodownload: true
    autodelete: true
  org.example.App.Codecs.Pack1:
    directory: extensions/codecs/pack1
    bundle: true
    no-autodownload: true
    autodelete: true
```

The contents of each `directory` are exported into that extension.

## Inspect loaded extensions

Inside a running Flatpak, inspect `/.flatpak-info` and look at the `Instance` group keys:

- `app-extensions`
- `runtime-extensions`

Use this when an extension is installed but not mounted; check branch, extension point name, directory existence, conditionals, and ID prefix.
