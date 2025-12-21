Extensions
==========

A brief overview of extensions can be found in :doc:`dependencies`. This
guide will go in depth on how to write an extension manifest and explain
the various properties used.

The first step on writing or loading an extension is to define an extension
point or use an existing one. The decision depends on where the extension
should be made available.

If for example it is a GTK/QT theme, it's best to define or use an
extension point in the runtime, so that it can be made available for
every application utilising said runtime.

If it is a specific tool eg. MPV plugins, which are only used by MPV,
it's best to define or use an extension point in the application. If
the tool is generic enough to be used by multiple unrelated applications,
the extension point should be defined in the runtime.

To see what are the available extension points in runtime or
applications, please see the corresponding flatpak or build
manifest.

``.Debug, .Locale, .Sources`` extensions created by Flatpak builder do
not need to be specified manually. These are automatically created and
loaded if installed.

Note that, ``.Locale`` extensions are by default only partially
installed (only for the configured languages) by Flatpak. To install the
full locale extension ``flatpak update --subpath= $FLATPAK_ID.Locale``
can be used.

Extension point
---------------

Now to define an extension point the ``add-extensions`` property can be
used in the manifest.

This is a typical example of an application extension point.

.. code-block:: yaml

  id: org.flatpak.app
  runtime: org.gnome.Platform
  runtime-version: '48'
  sdk: org.gnome.Sdk
  command: foo
  add-extensions:
    org.flatpak.app.plugin:
      version: '1.0'
      directory: extension_directory
      add-ld-path: lib
      merge-dirs: plug-ins;help
      subdirectories: true
      no-autodownload: true
      autodelete: true
  cleanup-commands:
    - mkdir -p ${FLATPAK_DEST}/extension_directory

The details of these properties are documented in the "Extension NAME"
section of :doc:`flatpak-command-reference`. Some of it is discussed
further below.

- ``org.flatpak.app.plugin`` is the name of the extension point.
  Extensions using this extension point must start their ID with the same
  prefix, for example ``org.flatpak.app.plugin.foo``.

- ``version`` is the branch to use when looking for the extension. This
  is useful to have parallel installations of the same extension in case
  of an API/ABI break. The value here will be the ``branch`` used in the
  extension manifest.

  If not specified it defaults to the ``runtime`` branch, that the
  extension point belongs to. So in the above example it is ``45``.

.. tip::
  ``versions`` is same as ``version`` but can be used for specifying
  multiple versions. For example, if an extension is published in the
  ``beta`` branch and the application is published in the ``stable``
  branch, to mount the extension in the application, the extension point
  in the application needs to have ``versions: 'stable;beta'``.

- ``directory`` is the path relative to prefix where everything is
  mounted by flatpak. For a runtime this will be relative to ``/usr``
  and for an application it will be relative to ``/app``. This supports
  a single value.

- ``add-ld-path`` adds a path relative to the extension prefix
  ``/app/extension_directory/foo/lib``, so that a library residing there
  can be loaded. This supports a single value. If the ``$LD_LIBRARY_PATH``
  environment variable exists, the path here is appended to it, otherwise
  it is put in a ld config file in ``/run/flatpak/ld.so.conf.d``.

- ``merge-dirs: plug-ins;help`` will merge the contents of these
  directories from multiple extensions using the same extension point.
  For example, if ``/app/extension_directory/foo/plug-ins`` and
  ``/app/extension_directory/bar/plug-ins`` exist, they will be merged
  and made available at ``/app/extension_directory/plug-ins``. Similarly
  for ``help``.

- ``subdirectories: true`` allows to have multiple extensions under the
  same ``directory``. So two extensions like
  ``org.flatpak.app.plugin.foo`` and ``org.flatpak.app.plugin.bar`` will
  be mounted at ``/app/extension_directory/foo`` and
  ``/app/extension_directory/bar``.

- ``no-autodownload: true`` will prevent installing all extensions
  under this extension point. This is the default.

- ``autodelete: true`` will remove all extensions under this extension
  point when the application is removed.

  ``.Debug`` and ``.Locale`` extensions are created with
  ``autodelete: true`` by default.

- ``mkdir -p ${FLATPAK_DEST}/extension_directory`` runs during the cleanup
  phase. It is used to initialise an empty directory for the extension to
  be loaded. This can also be done using ``post-install`` or ``build-commands``
  in a module. The directory has to end up in the final flatpak. If it is
  missing, there will be an error while running the application.

There is no extension property to extend the ``PATH``.

.. note::

  If the extension point is defined in a base runtime or SDK manifest
  ``add-extensions`` will make the extension point land in both
  ``.Sdk`` and ``.Platform``. If the extension point needs to be in only
  the child ``.Sdk`` but not in the child ``.Platform``
  ``inherit-sdk-extensions`` should be used which is discussed below.

There are other properties like ``download-if``, ``enable-if``,
``autoprune-unless`` etc. These are conditionals which must be ``true``
for the action to happen. These are typically not used in application
extension points.

An example of an extension point defined in runtime is the GL extension
point used in `Freedesktop SDK <https://gitlab.com/freedesktop-sdk/freedesktop-sdk/-/blob/1a8039407f8573725b16eab8779f2b0e1cd01629/elements/flatpak-images/platform.bst>`_
Freedesktop SDK uses `buildstream <https://buildstream.build/index.html>`_,
so the `format <https://docs.buildstream.build/master/format_project.html>`_
is different from the usual ``json`` or ``yaml`` format used by Flatpak
manifests.

.. code-block:: yaml

  Extension org.freedesktop.Platform.GL:
    # 1.4 is for Nvidia drivers
    versions: "%{branch};%{branch-extra};1.4"
    version: "1.4"
    directory: "%{lib}/GL"
    subdirectories: "true"
    no-autodownload: "true"
    autodelete: "false"
    add-ld-path: "lib"
    merge-dirs: "%{gl_merge_dirs}"
    download-if: "active-gl-driver"
    enable-if: "active-gl-driver"
    autoprune-unless: active-gl-driver

Most of this is already discussed above. Variables starting with ``%``
are private to the Freedesktop SDK. The version ``1.4`` is only used
for the proprietary NVIDIA drivers and is static since they have no
API/ABI guarantee.

``active-gl-driver`` is a flatpak conditional that is true if the name
of the active GL driver matches the extension point basename. The value
can be checked with ``flatpak --gl-drivers`` where ``host`` and
``default`` are always inserted. The command also looks at the
``FLATPAK_GL_DRIVERS`` environment variable and
``/sys/module/nvidia/version`` for nvidia kernel module version.

The ``default`` corresponds to a stable mesa fallback build whereas
``host`` is for `unmaintained` Flatpak extensions installed on host.

The resultant extension is called ``org.freedesktop.Platform.GL.default``
and it is downloaded and enabled automatically if ``active-gl-driver``
is true and deleted if only it is false.

The following conditionals are available: download-if, autoprune-unless
enable-if.

``download-if`` and ``enable-if`` supports the following:

- ``active-gl-driver``
- ``active-gtk-theme`` is true if the host GTK theme via ``org.gnome.desktop.interface``
  matches the extension basename.
- ``have-intel-gpu`` is true if the i915 kernel module is loaded.
- ``have-kernel-module-{module_name}`` is true if ``module_name`` is
  found in ``/proc/modules``.
- ``on-xdg-desktop-{desktop_name}`` is true if ``desktop_name``
  matches the value of ``XDG_CURRENT_DESKTOP`` on host.

``autoprune-unless`` supports only ``active-gl-driver``. If this evaluates
to ``false`` the extension will be considered unused and removed
automatically when doing ``flatpak uninstall --unused``.

Extensions are automatically mounted inside the runtime or app sandbox
provided the correct branch of the extension is installed, the
extension point for that extension is specified in the app or runtime
manifest and all the conditionals in the extension point match.

The list of loaded extensions can be found by inspecting the
``app-extensions`` and ``runtime-extensions`` keys of the ``Instance``
group in ``/.flatpak-info`` of a running Flatpak.

Loading existing extensions
---------------------------

This is a typical example of loading an existing extension
in the application. The extension is loaded at runtime and the user needs
to have it installed.

The extensions are mounted in alphabetical path order of directory.

.. warning::

  Some extensions are installed automatically by the runtime based on
  certain conditions and these do not need to be added to application
  manifests. Please see below for the purpose of extensions or
  extension points defined in the runtime. Similarly extensions created
  by Flatpak builder like ``.Locale, .Debug`` also do not need to be
  in application manifest.

Finding Base Runtime Version
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Some extensions are either built as part of the runtimes or have their
extension points defined within them. These typically follow the branch
pattern of the runtime. In case of
``org.gnome.{Platform, Sdk}, org.kde.{Platform, Sdk}`` they follow the
branch pattern of Freedesktop SDK (``org.freedesktop.{Platform, Sdk}``).

When using such extensions, the extension version must match the base
runtime version. This ensures ABI and API compatibility.

For example, ``org.freedesktop.Platform.ffmpeg-full`` is built as part
of the Freedesktop SDK and provides versions
``22.08, 23.08, 24.08, ...``. Suppose the application uses the runtime
``org.kde.Platform//5.15-24.08``, which is based on Freedesktop SDK.

To find the base runtime version of ``org.kde.Platform//5.15-24.08``,
run::

  flatpak remote-info flathub -m org.kde.Platform//5.15-24.08 | \
    grep -A 5 -F '[Extension org.freedesktop.Platform.GL]'

The output will have ``versions=24.08;24.08-extra;1.4``, and thus the
base runtime version is of ``org.kde.Platform//5.15-24.08`` is
``24.08``.

Similarly, for ``org.freedesktop.Sdk.Extension.texlive``, the extension
point ``org.freedesktop.Sdk.Extension`` is defined in the Freedesktop
SDK. To determine the base runtime version for a derived runtime such as
``org.gnome.Platform//48``, run::

  flatpak remote-info flathub -m org.gnome.Sdk//48 | \
    grep -A 5 -F '[Extension org.freedesktop.Sdk.Extension]'

The output will have ``versions=24.08``, and thus ``24.08`` needs
to be used as ``version`` in ``add-extensions``.

Examples
^^^^^^^^^

.. code-block:: yaml

  id: org.flatpak.cool-app
  runtime: org.kde.Platform
  runtime-version: '5.15-24.08'
  sdk: org.kde.Sdk
  command: foo
  add-extensions:
    org.freedesktop.Platform.ffmpeg-full:
      version: '24.08' # replace by appropriate version
      directory: lib/ffmpeg
      add-ld-path: .
  cleanup-commands:
    - mkdir -p ${FLATPAK_DEST}/lib/ffmpeg

.. code-block:: yaml

  id: org.flatpak.cool-app
  runtime: org.gnome.Platform
  runtime-version: '48'
  sdk: org.freedesktop.Sdk
  command: foo
  add-extensions:
    org.freedesktop.Sdk.Extension.texlive:
      directory: texlive
      version: '24.08' # replace by appropriate version
  cleanup-commands:
    - mkdir -p ${FLATPAK_DEST}/texlive

Note that ``Compat`` or ``GL32`` extensions need to specifically
requested. For providing runtime i386 support or for building i386
modules, please refer to :doc:`multiarch`.

A few related extension properties can be found in application or runtime
manifests. These are:

- ``inherit-extensions`` can be used to specify an extra set of extension
  points or extensions from the parent runtime or base that is inherited
  into the application or the current runtime. This for example, can be
  used to inherit i386 graphics drivers ``org.freedesktop.Platform.GL32``
  or ffmpeg ``org.freedesktop.Platform.ffmpeg-full`` in any application
  that uses the ``org.freedesktop.Platform`` runtime or a child runtime
  of it.

.. code-block:: yaml

  id: org.flatpak.cool-app
  runtime: org.gnome.Platform
  runtime-version: '48'
  sdk: org.gnome.Sdk
  base: org.winehq.Wine
  base-version: stable-24.08
  inherit-extensions:
    - org.freedesktop.Platform.GL32
    - org.freedesktop.Platform.ffmpeg-full
    - org.freedesktop.Platform.ffmpeg_full.i386
    - org.winehq.Wine.gecko
  command: foo

- ``add-build-extensions`` is same as ``add-extensions`` but the
  extensions are made available during build. This can be used to add
  build dependencies that reside in an extension based on the runtime
  being used.

  For example an application using the runtime
  ``org.freedesktop.Platform`` can use
  ``org.freedesktop.Sdk.Extension.openjdk11`` as a build-extension.

.. code-block:: yaml

  id: org.flatpak.cool-app
  runtime: org.freedesktop.Platform
  runtime-version: '24.08'
  sdk: org.freedesktop.Sdk
  add-build-extensions:
    - org.freedesktop.Sdk.Extension.openjdk11
  command: foo

- ``sdk-extensions`` can be used to install extra extensions having
  an extension point in the parent runtime that has to be installed for
  the app to build. These are similarly made available during build and
  not in the final flatpak. These always follow the branch pattern
  of the base runtime (see above).

.. code-block:: yaml

  id: org.flatpak.cool-app
  runtime: org.freedesktop.Platform
  runtime-version: '24.08'
  sdk: org.freedesktop.Sdk
  sdk-extensions:
    - org.freedesktop.Sdk.Extension.golang
  command: foo

- ``inherit-sdk-extensions`` is used to inherit extension points from the
  parent SDK into the child SDK. They aren't inherited into the child
  runtime. This is usually used when building runtimes or SDKs and not
  in applications.

.. code-block:: yaml

  inherit-sdk-extensions:
    - org.freedesktop.Sdk.Compat.i386
    - org.freedesktop.Sdk.Compat.i386.Debug

.. note::

  There is currently no way to add or inherit extensions per-arch. This
  means the extension should be available or made available for all the
  arches used by the application and vice-versa.

  This also means that certain extensions like i386 compatibility
  extensions like ``org.freedesktop.Sdk.Compat.i386`` should not be
  added to modules that build for ``aarch64``.

Extension manifest
------------------

Once the extension point is defined, an extension like
``org.flatpak.app.plugin.foo`` can be created.

This is a typical example of such an extension manifest. The explanation
is discussed below.

.. code-block:: yaml

  id: org.flatpak.app.plugin.foo
  branch: '1.0'
  runtime: org.flatpak.app
  runtime-version: 'stable'
  sdk: org.gnome.Sdk//48
  build-extension: true
  separate-locales: false
  build-options:
    prefix: /app/extension_directory/foo
    prepend-path: /app/extension_directory/foo/bin
    prepend-pkg-config-path: /app/extension_directory/foo/lib/pkgconfig
    prepend-ld-library-path: /app/extension_directory/foo/lib
  modules:
    - name: foo
      buildsystem: simple
      build-commands:
        - <build commands>
        - install -Dm644 org.flatpak.app.plugin.foo.metainfo.xml -t ${FLATPAK_DEST}/share/metainfo
      sources:
        ...

- ``id`` must have the correct prefix of the extension point.
- ``branch`` must be the ``version`` declared in the extension point.
- ``runtime`` should be the ID of the parent module where the extension
  point is defined.
- ``runtime-version`` is the version of the runtime used by the
  application. If the runtime is built locally and has not specified the
  ``branch`` property in its manifest, it defaults to ``master``,
  otherwise the value in ``branch`` is used.

  Applications on Flathub usually use either ``stable`` or ``beta``.
- ``sdk`` should be the same SDK used to build the runtime, followed by
  its version.
- ``build-extension: true`` instructs flatpak to build an extension.
- ``separate-locales: false`` disables creating a ``.Locale`` extension
  for this extension.

Flatpak-builder (>= 1.3.4), can compose metadata for extensions
automatically and it is no longer required to manually compose them
through commands in the manifest.

In case a manual compose is still required ``appstream-compose --basename=${FLATPAK_ID} --prefix=${FLATPAK_DEST} --origin=flatpak ${FLATPAK_ID}``
for composing with appstream-glib or ``appstreamcli compose --components=${FLATPAK_ID} --prefix=/ --origin=${FLATPAK_ID} --result-root=${FLATPAK_DEST} --data-dir=${FLATPAK_DEST}/share/app-info/xmls ${FLATPAK_DEST}`` for composing with appstreamcli can be used in ``build-commands``
or ``post-install`` along with having ``appstream-compose: false`` in
the top.

Note that the extension prefix or location of pkg-config files will not
be in ``$PATH`` or ``$PKG_CONFIG_PATH`` by default. Any such additional
variables need to be set in ``build-options``. This is done using
``prefix`` and ``prepend-*`` properties.

A MetaInfo file should be provided for discoverability in software
stores. This is a typical example of an extension MetaInfo file.

.. code-block:: xml

  <?xml version="1.0" encoding="UTF-8"?>
  <component type="addon">
    <id>org.flatpak.app.plugin.foo</id>
    <extends>org.flatpak.app</extends>
    <name>Foo</name>
    <summary>A nice summary</summary>
    <project_license>GPL-2.0-only</project_license>
    <metadata_license>CC0-1.0</metadata_license>
    <developer id="com.example">
      <name>Bar</name>
    </developer>
    <url type="homepage">https://flatpak.github.io/</url>
    <update_contact>bar_AT_example.org</update_contact>
    <releases>
      <release version="1.2.0" date="2023-12-03">
      <description>
        <p>Release description</p>
        <ul>
          <li>A release note</li>
          <li>A bugfix</li>
        </ul>
      </description>
      </release>
      <release version="1.0.0" date="2020-04-20"/>
    </releases>
  </component>

Bundled extensions
------------------

Extensions can also be built directly from the application manifest
instead of creating a separate extension manifest. The ``bundle: true``
property allows exporting them as separate extensions from the application
manifest. The extension needs to be defined in the application manifest
using ``add-extensions``. The contents of the ``directory`` will be
exported into that extension.

.. code-block:: yaml

  id: org.flatpak.cool-app
  runtime: org.kde.Platform
  runtime-version: '6.9'
  sdk: org.kde.Sdk
  command: foo
  add-extensions:
    org.flatpak.cool-app.codecs:
      directory: extensions/codecs
      subdirectories: true
      no-autodownload: true
      autodelete: true
    org.flatpak.cool-app.codecs.codec_pack1:
      directory: extensions/codecs/codec-pack1
      bundle: true
      no-autodownload: true
      autodelete: true
    org.flatpak.cool-app.codecs.codec_pack2:
      directory: extensions/codecs/codec-pack2
      bundle: true
      no-autodownload: true
      autodelete: true

Unmaintained Flatpak extensions
-------------------------------

Flatpak also supports `unmaintained extensions` that allows loading
extensions installed externally into ``/var/lib/flatpak/extension`` and
``$XDG_DATA_HOME/flatpak/extension`` from the host. This can be useful
to expose administrator policies, extensions, graphics drivers etc. to
Flatpak applications. The extension point of unmaintained extensions is
the same as above.

An example of an unmaintained extension can be found in browsers such as
`Chromium <https://github.com/flathub/org.chromium.Chromium/blob/dc7f731e7b62199a00bfa3ea3d123ff6d16936dc/org.chromium.Chromium.yaml>`_
or `Firefox <https://hg.mozilla.org/mozilla-central/diff/59e57f57dcb73a286822276d02f16e7b17018de6/taskcluster/docker/firefox-flatpak/runme.sh>`_
on Flathub.

The Firefox snippet translates to:

.. code-block:: yaml

  add-extensions:
    org.mozilla.firefox.systemconfig:
      directory: etc/firefox
      no-autodownload: true
  cleanup-commands:
    - mkdir -p ${FLATPAK_DEST}/etc/firefox

Now the required policy files for Firefox ``pref`` and ``policies.json``
can be placed in ``/var/lib/flatpak/extension/org.mozilla.firefox.systemconfig/x86_64/stable/defaults/pref``
and ``/var/lib/flatpak/extension/org.mozilla.firefox.systemconfig/x86_64/stable/policies/policies.json``
(or in ``$XDG_DATA_HOME/flatpak/extension/...``) respectively on host.
The path here is dependent on the extension point. These would appear
under ``/app/etc/firefox/policies/policies.json`` and
``/app/etc/firefox/defaults/pref`` inside the sandbox. (Firefox `supports <https://hg.mozilla.org/mozilla-central/file/23ee4ac2d048de0aac3fa27ce7eb0925c1903096/xpcom/io/SpecialSystemDirectory.cpp#l198>`_
reading policies from ``/app/etc``)

For details on Chromium, please look at the
`readme <https://github.com/flathub/org.chromium.Chromium>`_.

Creating an unmaintained Gtk theme extension
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The following script can be used to create an unmaintained extension
for the host's Gtk 3 theme. This is useful when the theme is not packaged
as an extension in a remote.

The script expects the theme to be installed in ``/usr/share/themes``
or ``$XDG_DATA_HOME/themes``.

.. code-block:: bash

  #!/bin/sh

  DEFAULT_ARCH=$(flatpak --default-arch)
  THEME_NAME=$(gsettings get org.gnome.desktop.interface gtk-theme | tr -d "'")
  XDG_DATA_HOME=${XDG_DATA_HOME:-$HOME/.local/share}
  THEME_EXTENSION_DIR=$XDG_DATA_HOME/flatpak/extension/org.gtk.Gtk3theme.$THEME_NAME/$DEFAULT_ARCH/3.22

  mkdir -p "$THEME_EXTENSION_DIR"

  if [ -d "/usr/share/themes/$THEME_NAME/gtk-3.0/" ]; then
  	cp -r /usr/share/themes/"$THEME_NAME"/gtk-3.0/* "$THEME_EXTENSION_DIR"
  elif [ -d "$XDG_DATA_HOME/themes/$THEME_NAME/gtk-3.0/" ]; then
  	cp -r "$XDG_DATA_HOME"/themes/"$THEME_NAME"/gtk-3.0/* "$THEME_EXTENSION_DIR"
  else
  	echo "Could not find theme files"
  	rmdir --ignore-fail-on-non-empty "$THEME_EXTENSION_DIR"
  	exit 1
  fi

Extensions or extension points defined by runtime
-------------------------------------------------

The following extensions and extension points are defined in the
Freedesktop runtime/SDK or are shipped along with it. Most of these
are inherited by the GNOME and KDE runtimes as well. These may
change over time, please check the respective project.

These are common to the Freedesktop SDK and runtime:

- ``org.freedesktop.Platform.GL`` – Extension for graphics drivers managed
  by the runtime and installed or removed automatically. The default
  has two branches ``${RUNTIME_VERSION}`` and
  ``${RUNTIME_VERSION}-extra``, the latter containing support for
  patented codecs.

  ``org.freedesktop.Platform.GL.Debug`` – Debug extension point for
  ``org.freedesktop.Platform.GL``, managed by the runtime but the user needs
  to explicitly install ``org.freedesktop.Platform.GL.Debug.default//${RUNTIME_VERSION}``
  and ``org.freedesktop.Platform.GL.Debug.default//${RUNTIME_VERSION}-extra``
  to have the debug symbols available.

  The following extensions utilise the above two extension points::

    org.freedesktop.Platform.GL.default//${RUNTIME_VERSION}
    org.freedesktop.Platform.GL.default//${RUNTIME_VERSION}-extra
    org.freedesktop.Platform.GL.Debug.default//${RUNTIME_VERSION}
    org.freedesktop.Platform.GL.Debug.default//${RUNTIME_VERSION}-extra

    org.freedesktop.Platform.GL32.default//${RUNTIME_VERSION}
    org.freedesktop.Platform.GL32.default//${RUNTIME_VERSION}-extra
    org.freedesktop.Platform.GL32.Debug.default//${RUNTIME_VERSION}
    org.freedesktop.Platform.GL32.Debug.default//${RUNTIME_VERSION}-extra

    org.freedesktop.Platform.GL.nvidia-${DRIVER_VERSION}
    org.freedesktop.Platform.GL32.nvidia-${DRIVER_VERSION}

- ``org.freedesktop.Platform.VulkanLayer`` – Extension point for
  `Vulkan layers <https://github.com/KhronosGroup/Vulkan-Guide/blob/main/chapters/layers.md>`_.
  Developers can provide extensions using this extension point
  and the user needs to install those extensions to have them available.
- ``org.freedesktop.Platform.GStreamer`` – Extension point for GStreamer
  plugins. Developers can provide extensions using this extension point
  and the user needs to install those extensions to have them available.
- ``org.freedesktop.Platform.Icontheme`` – Extension point for icon themes.
  Developers can provide extensions using this extension point
  and the user needs to install those extensions to have them available.
- ``org.gtk.Gtk3theme`` – Extension point for Gtk3 themes. Extensions
  under this extension point are automatically installed by Flatpak
  if an extension matching the host theme is available. Developers can
  provide extensions using this extension point.
- ``org.freedesktop.Platform.VAAPI.Intel`` – Extension providing Intel
  VAAPI media drivers. This is automatically installed if the user
  has an Intel GPU. This has a compat i386 extension
  ``org.freedesktop.Platform.VAAPI.Intel.i386``.
- ``org.freedesktop.Platform.VAAPI.nvidia`` – Extension providing Nvidia
  VAAPI media drivers. This is automatically installed if the user
  has an Nvidia GPU. This has a compat i386 extension
  ``org.freedesktop.Platform.VAAPI.nvidia.i386``.
- ``org.freedesktop.Platform.openh264`` – Extension providing OpenH264,
  automatically installed by the runtime. (This is discontinued since
  Freedesktop SDK 25.08)
- ``org.freedesktop.Platform.ffmpeg-full`` – Extension providing ffmpeg
  with support for patented codecs. This needs to explicitly added to the
  manifest using ``add-extensions`` by the developer, so that it becomes
  available when the user installs it.This has a compat i386 extension
  ``org.freedesktop.Platform.ffmpeg_full.i386``. (This is discontinued
  since Freedesktop SDK 25.08)
- ``org.freedesktop.Platform.codecs-extra`` – This is the successor
  to ``org.freedesktop.Platform.ffmpeg-full`` available since
  Freedesktop SDK 25.08. This is a runtime extension and it will be
  automatically installed by the runtime when installing an app. This
  has a compat i386 extension
  ``org.freedesktop.Platform.codecs_extra.i386``. The branch is set to
  ``$freedesktop-sdk-version-extra`` i.e. ``25.08-extra, 26.08-extra``
  and so on. Users and distributors who want to block or not use
  patented codecs can use ``flatpak mask`` to mask this extension or
  the ``-extra`` branch pattern.

These are only in Freedesktop SDK:

- ``org.freedesktop.Sdk.Extension`` – Extension point for SDK extensions
  like extra toolchains (eg. LLVM), compilers and language specific
  tools to aid building applications or provide language support for
  development tools such as IDEs.

  SDK extensions available on Flathub are listed
  `here <https://github.com/orgs/flathub/repositories?q=org.freedesktop.Sdk.Extension.>`_.

  The application developer needs to explicitly add these extensions
  in the manifest by using ``sdk-extensions`` when building an app.

Extensions marked as ``Compat`` in the name or ``GL32`` provide compat
support for extra architectures and needs to explicitly requested.

Additionally all SDKs provide a ``.Docs`` extension for documentation.
