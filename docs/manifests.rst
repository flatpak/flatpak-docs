Manifests
=========

The input to ``flatpak-builder`` is a JSON or YAML file that describes the
parameters for building an application, as well as instructions for each of
the modules that are to be built. This file is called the manifest.

This page provides information and guidance on how to use manifests, including
an explanation of the most common parameters that can be specified. It is
recommended to have followed the :doc:`first-build` tutorial before reading
this section, and to be familiar with :doc:`flatpak-builder`.

Manifest files should be named using the application ID. For example, the
manifest file for GNOME Dictionary is named ``org.gnome.Dictionary.yml``. This
page uses this manifest file for all its examples.

A complete list of all the properties that can be specified in manifest files
can be found in the :doc:`flatpak-builder-command-reference`, as well as the
``flatpak-manifest`` man page.

Basic properties
----------------

Each manifest file should specify basic information about the application that
is to be built, including the ``id``, ``runtime``, ``runtime-version``,
``sdk`` and ``command`` parameters. These properties are typically specified
at the beginning of the file.

For example, the GNOME Dictionary manifest includes:

.. code-block:: yaml

  id: org.gnome.Dictionary
  runtime: org.gnome.Platform
  runtime-version: '48'
  sdk: org.gnome.Sdk
  command: gnome-dictionary

Specifying a runtime and runtime version allows that the runtime that is
needed by your application to be automatically installed on users' systems.

File renaming
-------------

Exports are application files that are made available to the host, and include
things like the application's ``.desktop`` file and icon.

The names of files that are exported by a Flatpak must be prefixed using the
application ID, such as ``org.gnome.Dictionary.desktop``. The best way to
do this is to rename these files directly in the application's source.

If renaming exported files to use the application ID is not possible,
``flatpak-builder`` allows them to be renamed as part of the build
process. This can be done by specifying one of the following properties in
the manifest:

- ``rename-icon`` - rename the application icon
- ``rename-desktop-file`` - rename the ``.desktop`` filename
- ``rename-appdata-file`` - rename the MetaInfo file

Each of these properties accepts the name of the source file to be
renamed. ``flatpak-builder`` then automatically renames the file to match
the application ID. Note that this renaming method can introduce internal
naming conflicts, and that renaming files in tree is therefore the most
reliable approach.

Finishing
---------

Applications that are run with Flatpak have extremely limited access to the
host environment by default, but applications require access to resources
outside of their sandbox in order to be useful. Finishing is the build stage
where the application's sandbox permissions are specified, in order to give
access to these resources.

The finishing manifest section uses the ``finish-args`` property, which can
be seen in the Dictionary manifest file:

.. code-block:: yaml

  finish-args:
    # X11 + XShm access
    - --share=ipc
    - --socket=fallback-x11
    # Wayland access
    - --socket=wayland
    # GPU acceleration if needed
    - --device=dri
    # Needs to talk to the network:
    - --share=network
    # Needs to save files locally
    - --filesystem=xdg-documents

Guidance on which permissions to use can be found in the
:doc:`sandbox-permissions`.

Cleanup
-------

The cleanup property can be used to remove files produced by the build process
that are not wanted as part of the application, such as headers or developer
documentation. Two properties in the manifest file are used for this.
This can be either done for each modules in which case only names created
by that module will be matched or at the toplevel which will match
anything created in the entire manifest.

Items starting with `/` are taken to be relative to the prefix, so
``/include`` will cleanup ``/app/include``, otherwise it matches the
basename.

First, a list of filename patterns can be included::

  cleanup:
    - '/include'
    - '/bin/foo-*'
    - '*.a'

A cleanup with ``*``, at the `module level` will cleanup all artifacts
built from that module. This is often useful for build dependencies of
a module that does not need to be shipped in the final Flatpak package::

  cleanup:
    - '*'

The `cleanup-commands` property can be a list of cleanup commands::

  cleanup-commands:
    - 'find /app/bin -mindepth 1 -maxdepth 1  -name 'rpm*' ! -name 'rpm2cpio' -delete'

Note that, instead of cleaning up unnecessary files, it is often better
to build less components through ``config-opts, build-commands, make-args``.
For example, if the application does not need documentation files or
manpages, it's best to stop building them. This should make the build
faster in some cases and reduce the need for excessive cleanups.

Modules
-------

The module list specifies each of the modules that are to be built as part
of the build process. One of these modules is the application itself, and
other modules are dependencies and libraries that are bundled as part of
the Flatpak. While simple applications may only specify one or two modules,
and therefore have short modules sections, some applications can bundle
numerous modules and therefore have lengthy modules sections.

Modules are built in the order they are declared in the manifest. If any
module changes, that module and all the subsequent modules below it will
be rebuilt, otherwise it should use the cache.

The general recommendation is to place the "main" module, usually the
module for the main application as the last module in the manifest but
if there is a module which gets updated often and is independent from the
rest, that module can also be placed as the last module to avoid
rebuilding everything else.

Modules can either be nested to clearly show the dependency structure
or be linearly declared.

.. code-block:: yaml

  # Nested

  finish-args:
    - --share=ipc
    - --socket=fallback-x11
    - --socket=wayland
    - --socket=pulseaudio

    modules:
      - name: video-player-app
        buildsystem: meson
        config-opts:
          - --buildtype=release
        cleanup:
          - /share/man
        sources:
          - type: archive
            url: https://example.com/release.tar.gz
            sha256: 216656c4495bb3ca02dc4ad9cf3da8e8f15c8f80e870eeac8eb1eedab4c3788b
        modules:
          - name: libmpv
            buildsystem: meson
            config-opts:
              - -Dlibmpv=true
            sources:
              - type: archive
                url: https://example.com/mpv.tar.gz
                sha256: 2ca92437affb62c2b559b4419ea4785c70d023590500e8a52e95ea3ab4554683
            modules:
              - "shared-modules/lua5.1/lua-5.1.5.json"

              - name: libv4l2
                buildsystem: meson
                sources:
                  - type: archive
                    url: url: https://example.com/libv4l2.tar.gz
                    sha256: 0fa075ce59b6618847af6ea191b6155565ccaa44de0504581ddfed795a328a82
  # Linear

  finish-args:
    - --share=ipc
    - --socket=fallback-x11
    - --socket=wayland
    - --socket=pulseaudio

    modules:
      - "shared-modules/lua5.1/lua-5.1.5.json"

      - name: libv4l2
        buildsystem: meson
        sources:
          - type: archive
            url: url: https://example.com/libv4l2.tar.gz
            sha256: 0fa075ce59b6618847af6ea191b6155565ccaa44de0504581ddfed795a328a82

      - name: libmpv
        buildsystem: meson
         config-opts:
           - -Dlibmpv=true
        sources:
          - type: archive
            url: https://example.com/mpv.tar.gz
            sha256: 2ca92437affb62c2b559b4419ea4785c70d023590500e8a52e95ea3ab4554683

      - name: video-player-app
        buildsystem: meson
        config-opts:
          - --buildtype=release
        cleanup:
          - /share/man
        sources:
          - type: archive
            url: https://example.com/release.tar.gz
            sha256: 216656c4495bb3ca02dc4ad9cf3da8e8f15c8f80e870eeac8eb1eedab4c3788b

As can be seen, each listed module has a ``name`` (which can be freely
assigned) and a list of ``sources``. Each source has a ``type``, and available
types include:

 - ``archive`` - ``.tar`` or ``.zip`` archive files
 - ``git`` - Git repositories
 - ``bzr`` - Bazaar repositories
 - ``file`` - local/remote files (these are copied into the source directory)
 - ``dir`` - local directories (these are copied into the source directory)
 - ``script`` - an array of shell commands (these are put in a shellscript
   file)
 - ``shell`` - an array of shell commands that are run during source extraction
 - ``patch`` - a patch (are applied to the source directory)
 - ``extra-data`` - data that can be downloaded at install time; this can
   include archive or package files

Different properties are available for each source type, which are listed
in the :doc:`module-sources`.

Supported build systems
```````````````````````

Modules can be built with a variety of build systems, including:

- `autotools <https://www.gnu.org/software/automake/manual/html_node/Autotools-Introduction.html>`_
- `cmake <https://cmake.org/>`_
- `cmake-ninja <https://cmake.org/cmake/help/v3.0/generator/Ninja.html>`_
- `meson <https://mesonbuild.com/>`_
- `qmake <https://doc.qt.io/archives/qt-5.15/qmake-overview.html>`_

A "simple" build method is also available, which allows a series of
commands to be specified.

Each of the above buildsystem sets up the installation prefix, libdir
etc. and runs a series of commands to configure (``meson setup``, or
``./autogen.sh, ./configure`` or ``cmake``), build, install
(``ninja install, make install`` etc.) and optionally run tests
(``make check, ninja test`` etc.).

Thus they can also be achieved by using the ``simple`` buildsystem and
manually specifying the commands.

.. note::

  Using the proper buildsystem is almost always preferred rather than
  manually handling the correct setup.

An example is provided below.

.. code-block:: yaml

  # Using autotools without configure
  - name: ffnvcodec
    buildsystem: autotools
    no-autogen: true
    make-install-args:
      - PREFIX=/app
    sources:
      - type: git
        url: https://github.com/FFmpeg/nv-codec-headers.git
        commit: 43d91706e097565f57b311e567f0219838bcc2f6
        tag: n11.1.5.3

  # Using meson buildsystem
  - name: libdrm
    buildsystem: meson
    builddir: true
    config-opts:
      - -Dtests=false
    sources:
      - type: git
        url: https://gitlab.freedesktop.org/mesa/drm.git
        tag: libdrm-2.4.124

  # Using simple

  - name: ffnvcodec
    buildsystem: simple
    build-commands:
      - make -j$FLATPAK_BUILDER_N_JOBS PREFIX=/app install
    sources:
      - type: git
        url: https://github.com/FFmpeg/nv-codec-headers.git
        commit: 43d91706e097565f57b311e567f0219838bcc2f6
        tag: n11.1.5.3

  - name: libdrm
    buildsystem: simple
    build-commands:
      - meson setup builddir --prefix=/app --libdir=/app/lib -Dtests=false
      - ninja -C builddir install
    sources:
      - type: git
        url: https://gitlab.freedesktop.org/mesa/drm.git
        tag: libdrm-2.4.124

Shared Modules
``````````````

Flathub contains a `shared modules <https://github.com/flathub/shared-modules>`_
repository containing build manifests for commonly used modules. These
are usally shared by apps on Flathub and maintained in a single place.
The repository is intended to be used as a git submodule.

Please see the `readme <https://github.com/flathub/shared-modules/blob/master/README.md>`_
for details on how to use this.

Flatpak Builder Tools
`````````````````````

`Flatpak Builder Tools (or flatpak-builder-tools) <https://github.com/flatpak/flatpak-builder-tools>`_ is a collection of scripts to aid using `flatpak-builder`. In this repository, each directory contains instructions to generate a manifest for the respective platform.

Example manifests
-----------------

Flathub hosts a large collection of applications and the respective
manifests can be browsed and searched via `GitHub <https://github.com/flathub>`_.
