---
name: flatpak-packaging
description: Build, configure, debug, and publish Flatpak applications with flatpak-builder manifests. Use this skill when the user asks you to create or edit Flatpak YAML/JSON manifests, choose runtimes and SDKs, configure finish-args sandbox permissions, add module sources or dependencies, package Python/Electron/.NET/Qt apps, build/install/test with flatpak-builder, inspect or debug sandboxes, create bundles or repositories, write .flatpakref/.flatpakrepo files, or advise on Flatpak app IDs, desktop files, AppStream metadata, extensions, and Flathub-style packaging.
---

# Flatpak Packaging

## Overview

Use this skill to turn an application into a Flatpak package or to maintain an existing Flatpak manifest. Prefer the project's own build system and upstream metadata, keep sandbox permissions minimal, and verify with `flatpak-builder` whenever the local environment has Flatpak tooling installed.

## First Moves

1. Inspect the project before writing a manifest: application ID, build system, runtime/toolkit, executable command, desktop file, MetaInfo/AppStream file, icons, network/filesystem/device needs, and distribution target.
2. Check for an existing manifest such as `*.yml`, `*.yaml`, `*.json`, or Flathub packaging. Preserve local style unless it conflicts with Flatpak requirements.
3. Choose the narrowest runtime that already contains the app's major toolkit. Use Freedesktop for generic apps, GNOME for GNOME/GTK apps, KDE for Qt/KDE apps, and matching SDK branches for builds.
4. Put the main app module last unless a frequently changing independent module should intentionally be last to improve cache reuse.
5. Pin downloaded sources with checksums and Git sources with exact commits. Avoid branch-only Git sources for reproducible builds.
6. Prefer portals and XDG base directories over broad filesystem or D-Bus permissions.

## Reference Routing

- Read `references/manifest-and-sources.md` when creating or changing manifest structure, modules, build systems, cleanup, source types, or bundled dependencies.
- Read `references/runtimes-and-conventions.md` when choosing runtime/SDK branches or checking app IDs, desktop files, icons, MetaInfo, exported files, D-Bus service files, MIME files, and XDG paths.
- Read `references/sandbox-permissions.md` when deciding `finish-args`, reducing permissions, using portals, debugging D-Bus access, granting filesystem/device access, or using conditional permissions.
- Read `references/build-debug.md` when building, installing locally, entering a sandbox, using GDB/strace/valgrind/perf, auditing bus traffic, testing portal permissions, or troubleshooting a failed build/run.
- Read `references/framework-notes.md` for Python, Electron, .NET, or Qt-specific manifest patterns and generator tooling.
- Read `references/distribution-and-repositories.md` when creating bundles, repositories, `.flatpakref` or `.flatpakrepo` files, signing, hosting, or publishing updates.
- Read `references/extensions.md` when defining extension points, consuming SDK/runtime extensions, building app plugins/add-ons, or using bundled extensions.

## Core Workflow

1. Create or edit the manifest:

```yaml
id: org.example.App
runtime: org.freedesktop.Platform
runtime-version: '25.08'
sdk: org.freedesktop.Sdk
command: app-command
finish-args:
  - --share=ipc
  - --socket=fallback-x11
  - --socket=wayland
  - --device=dri
modules:
  - name: app
    buildsystem: meson
    sources:
      - type: git
        url: https://example.org/app.git
        tag: v1.0.0
        commit: 0123456789abcdef0123456789abcdef01234567
```

2. Build and install locally:

```bash
flatpak remote-add --if-not-exists --user flathub https://dl.flathub.org/repo/flathub.flatpakrepo
flatpak-builder --force-clean --user --install-deps-from=flathub --repo=repo --install builddir org.example.App.yml
flatpak run org.example.App
```

3. Validate exported desktop metadata:

```bash
desktop-file-validate path/to/org.example.App.desktop
appstreamcli validate --explain path/to/org.example.App.metainfo.xml
```

4. Debug inside the sandbox when runtime behavior differs from the host:

```bash
flatpak run --command=sh --devel --filesystem="$(pwd)" org.example.App
flatpak-builder --run builddir org.example.App.yml sh
flatpak run --log-session-bus org.example.App
```

5. Package for distribution only after the app installs, launches, and has correct metadata. Repositories are preferred for updates; single-file bundles are for direct sharing or removable media.

## Permission Stance

- Treat `finish-args` as static packaging policy; variables are not expanded inside these arguments.
- Avoid `--filesystem=home`, `--filesystem=host`, `--socket=session-bus`, `--socket=system-bus`, and `--device=all` unless the app is a development/system tool or no narrower permission works.
- For GUI apps that support Wayland, use `--socket=wayland` plus `--socket=fallback-x11`; for X11-only apps, use `--socket=x11` and usually `--share=ipc`.
- Use `:ro` for read-only filesystem needs, XDG directory grants for specific user folders, and `--persist=DIR` for legacy hardcoded dot-directories.
