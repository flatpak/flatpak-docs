# Manifest and Sources

Related public docs: [Manifests](https://docs.flatpak.org/en/latest/manifests.html), [Module Sources](https://docs.flatpak.org/en/latest/module-sources.html), [Dependencies](https://docs.flatpak.org/en/latest/dependencies.html), [Building your first Flatpak](https://docs.flatpak.org/en/latest/first-build.html).

## Manifest essentials

A Flatpak manifest is JSON or YAML consumed by `flatpak-builder`. Name the file after the app ID, for example `org.example.App.yml`.

Required or common top-level keys:

- `id`: unique app ID in reverse-DNS style.
- `runtime`: runtime ID, such as `org.freedesktop.Platform`, `org.gnome.Platform`, or `org.kde.Platform`.
- `runtime-version`: branch of the runtime, quoted when numeric-looking.
- `sdk`: matching SDK, usually the runtime ID with `.Sdk`.
- `command`: executable in `/app/bin` (or otherwise on PATH inside the sandbox).
- `finish-args`: sandbox permissions applied during build finish.
- `modules`: ordered list of dependencies and the app itself.
- `sdk-extensions`, `build-options`, `cleanup`, `cleanup-commands`, `rename-*`, `add-extensions`: common optional keys.

Minimal example:

```yaml
id: org.example.App
runtime: org.freedesktop.Platform
runtime-version: '25.08'
sdk: org.freedesktop.Sdk
command: app
modules:
  - name: app
    buildsystem: simple
    build-commands:
      - install -Dm755 app.sh /app/bin/app
    sources:
      - type: script
        dest-filename: app.sh
        commands:
          - echo "Hello from a sandbox"
```

## Modules

Modules are built in manifest order. If a module changes, that module and later modules are rebuilt from cache. Put stable dependencies first and the main app module last unless a frequently changing independent module should be last.

Use the native `buildsystem` when available:

- `meson`: prefer for Meson projects; add `builddir: true` when needed.
- `cmake-ninja`: prefer over hand-written CMake commands.
- `cmake`, `autotools`, `qmake`: use when the upstream build system matches.
- `simple`: use for custom installs, binary repackaging, wrapper scripts, generated sources, or build systems not covered by flatpak-builder.

Avoid replacing a supported build system with `simple` unless the project genuinely needs custom commands. The supported build systems set prefixes, libdirs, build/install commands, and test hooks correctly.

Common module keys:

- `name`: local module name.
- `buildsystem`: build driver.
- `config-opts`: build configuration options.
- `build-commands`: shell commands for `simple` modules.
- `make-args`, `make-install-args`, `no-autogen`, `builddir`, `subdir`: build customization.
- `sources`: sources applied before build.
- `modules`: nested modules, useful when representing dependency structure.
- `cleanup`: module-level cleanup; `'*'` removes all artifacts from that module, useful for build-only helpers.

## Sources

Most source types support `path` for local files relative to the manifest root and `url` for remote downloads. Remote `url` sources generally require `sha256` or `sha512`.

Common source keys:

- `only-arches` and `skip-arches`: download/build source only for selected architectures.
- `dest`: place a source under a subdirectory in the build source tree.
- `dest-filename`: set a predictable downloaded/copied filename.

Use these source types:

- `archive`: tar/zip/rpm/7z archives. Include `url`, checksum, optional `mirror-urls`, `strip-components`, `dest-filename`, and `archive-type` if suffix detection fails.
- `git`: Git repository. Include `url` and exact `commit`; `tag` is fine with `commit`. Avoid branch-only sources for reproducibility. `path` can point to a local Git repo. `disable-shallow-clone` and `disable-submodules` alter defaults.
- `file`: copy a local or remote file unchanged. Often paired with `simple` commands and `dest-filename`.
- `dir`: copy a local directory. Useful in CI/local builds, but avoid for store submissions because it bypasses source caching and reproducibility.
- `patch`: apply patch files. Use `path` or `paths`; default is `patch -p 1`; `strip-components`, `options`, `use-git`, and `use-git-am` adjust behavior.
- `shell`: run shell commands during source extraction for small fail-safe tweaks. Use patches for complex changes.
- `script`: generate a `/bin/sh` script source with `commands` and `dest-filename`, commonly for app wrappers.
- `extra-data`: download proprietary or very large data at install time. Requires `url`, `sha256`, `size`, and `filename`, plus an installed `/app/bin/apply_extra` script.

## Dependencies

Prefer dependencies already in the chosen runtime. Bundle only what the runtime lacks, what must be a different version, what needs patches, or app-specific data/resources.

Before hand-writing many modules, check for:

- Flathub shared modules for common native dependencies.
- `flatpak-builder-tools` generators for ecosystems such as Python, Node/Electron, and .NET.
- BaseApps for large frameworks, especially Electron.
- SDK extensions for build-time tools such as Node.js, Go, Java, or .NET.

## Cleanup

Use top-level `cleanup` for files produced anywhere in the manifest and module-level `cleanup` for files produced by one module. Paths beginning with `/` are relative to `/app`, so `/include` removes `/app/include`. Prefer configuring builds to not produce docs, static libs, or headers instead of cleaning them afterwards.

Common cleanup:

```yaml
cleanup:
  - /include
  - /share/man
  - '*.a'
```

## Exported files and renaming

Desktop files, icons, and MetaInfo files exported to the host should be named with the app ID. Prefer fixing names in upstream source. If that is not possible, `flatpak-builder` can rename:

```yaml
rename-icon: old-icon-name
rename-desktop-file: old.desktop
rename-appdata-file: old.appdata.xml
```

Use renaming as a compatibility escape hatch; direct app-ID filenames are more reliable.

## Useful build variables

- `$FLATPAK_ID`: app ID.
- `$FLATPAK_DEST`: install prefix, usually `/app` for apps.
- `$FLATPAK_BUILDER_N_JOBS`: parallel job count for build commands.

Do not assume these variables exist at runtime or inside `apply_extra`; they are build-time variables.
