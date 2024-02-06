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

Extension point
---------------

Now to define an extension point the ``add-extensions`` property can be
used in the manifest.

This is a typical example of an application extension point.

.. code-block:: yaml

  id: org.flatpak.app
  runtime: org.gnome.Platform
  runtime-version: '45'
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
  of an API/ABI break. This will be the ``branch`` used in the
  extension manifest.

  If not specified it defaults to application or runtime branch of the
  extension point.

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
  can be loaded. This supports a single value.

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

For a list of this conditionals, please see ``man flatpak-metadata``.

Loading existing extensions
---------------------------

This is a typical example of loading an existing extension
in the application. The extension is loaded at runtime and the user needs
to have it installed.

The extensions are mounted in alphabetical path order of directory.
ld-path prepended before ``/app/lib`` for app extensions and appended
after ``/app/lib`` in case of runtime extensions.

``org.freedesktop.Platform.ffmpeg-full`` is an extension of the runtime
``org.freedesktop.Platform`` and ``org.kde.Platform`` is a child runtime
of ``org.freedesktop.Platform``.

.. code-block:: yaml

  id: org.flatpak.cool-app
  runtime: org.kde.Platform
  runtime-version: '5.15-23.08'
  sdk: org.kde.Sdk
  command: foo
  add-extensions:
    org.freedesktop.Platform.ffmpeg-full:
      version: '23.08'
      directory: lib/ffmpeg
      add-ld-path: .
  cleanup-commands:
    - mkdir -p ${FLATPAK_DEST}/lib/ffmpeg

``org.freedesktop.Sdk.Extension`` is an extension of the SDK
``org.freedesktop.Sdk``.

.. code-block:: yaml

  id: org.flatpak.cool-app
  runtime: org.freedesktop.Platform
  runtime-version: '23.08'
  sdk: org.freedesktop.Sdk
  command: foo
  add-extensions:
    org.freedesktop.Sdk.Extension.texlive:
      directory: texlive
      version: '22.08'
  cleanup-commands:
    - mkdir -p ${FLATPAK_DEST}/texlive

There is currently no way to `request` autodownload of a runtime
extension from an application. The extension point in the runtime has
to be set to autodownload or the user has to manually install it.

A few related extension properties can be found in application or runtime
manifests. These are:

- ``inherit-extensions`` can be used to specify an extra set of extensions
  having an extension point in the parent runtime or base that is inherited
  in the application. This for example, can be used to inherit i386
  graphics drivers ``org.freedesktop.Platform.GL32`` or ffmpeg
  ``org.freedesktop.Platform.ffmpeg-full`` in any application that
  uses the ``org.freedesktop.Platform`` runtime or a child runtime of it.

.. code-block:: yaml

  id: org.flatpak.cool-app
  runtime: org.gnome.Platform
  runtime-version: '45'
  sdk: org.gnome.Sdk
  base: org.winehq.Wine
  base-version: stable-23.08
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
  runtime-version: '23.08'
  sdk: org.freedesktop.Sdk
  add-build-extensions:
    - org.freedesktop.Sdk.Extension.openjdk11
  command: foo

- ``sdk-extensions`` can be used to install extra extensions having
  extension point in the parent runtime that has to be installed for the
  app to build. These are similarly made available during build and
  not in the final flatapk.

.. code-block:: yaml

  id: org.flatpak.cool-app
  runtime: org.freedesktop.Platform
  runtime-version: '23.08'
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
  sdk: org.gnome.Sdk//45
  build-extension: true
  separate-locales: false
  appstream-compose: false
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
        - appstream-compose --basename=${FLATPAK_ID} --prefix=${FLATPAK_DEST} --origin=flatpak ${FLATPAK_ID}
      sources:
        ...

- ``id`` must have the correct prefix of the extension point.
- ``branch`` refers to the extension version.
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
- ``appstream-compose: false`` disables running ``appstream-compose``.
  This is run manually in the build or cleanup stage.

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
      <project_license>GPL-2.0</project_license>
      <metadata_license>CC0-1.0</metadata_license>
      <developer_name>Bar</developer_name>
      <url type="homepage">https://flatpak.github.io/</url>
      <update_contact>bar_AT_example.org</update_contact>
      <releases>
      <release version="1.2.0" date="2023-12-03">
        <description>
        <ul>
          <li>A release note</li>
          <li>A bugfix</li>
        </ul>
      </description>
      <release version="1.0.0" date="2020-04-20"/>
     </releases>
    </component>

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
