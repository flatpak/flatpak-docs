Module Sources
==============

This page will go in depth on each source type with examples.

Sources are copied or downloaded to the source directory before the build
starts. The source directory lives under flatpak-builder's state
directory, which by default is located at ``.flatpak-builder`` on host,
relative to the manifest root. The source directory for bzr, git and svn
sources is at ``.flatpak-builder/git, .flatpak-builder/bzr, .flatpak-builder/svn``
respectively while ``.flatpak-builder/downloads/<checksum>`` is used
for file and archive sources.

The ``type`` property of a source determines how that source will be
handled before or during build by ``flatpak`` or ``flatpak-builder``.

Most of the sources support ``path`` and ``url`` property where ``path``
is relative to the root of manifest and ``url`` is a direct url to the
source.

The ``url`` property requires a checksum to be specified after
using ``sha256`` or ``sha512``.

There are a few properties common to all source types ``only-arches``,
``skip-arches`` and ``dest``.

``only-arches`` and ``skip-arches`` are used to build the source only
on the specified architecture or skip building it on the specified
architecture respectively. This is often useful for binary packages.

.. note::

  Note that ``only-arches`` or ``skip-arches`` are used to control if
  that particular source is downloaded on the specified architecture(s).
  If not specified, the default is to download on all architectures.

.. code-block:: yaml

  modules:
    - name: foo
      buildsystem: simple
      build-commands:
        - ...
        - ...
      sources:
        - type: file
          url: https://example.org/foo-1.0-x86_64.deb
          sha256: 26c9007c655483438d83f17070190fc4b6bccb8ff5efc30fcedbb7d162681a7f
          dest-filename: foo.deb
          only-arches:
            - x86_64
        - type: file
          url: https://example.org/foo-1.0-aarch64.deb
          sha256: 26c9007c655483438d83f17070190fc4b6bccb8ff5efc30fcedbb7d162681a7f
          dest-filename: foo.deb
          only-arches:
            - aarch64

``dest`` is used to indicate a directory inside the source directory
where the given source will be placed:

.. code-block:: yaml

  modules:
    - name: coolapp
      buildsystem: simple
      build-commands:
        - ...
      sources:
        - type: archive
          url: https://example.org/foo.tar.xz
          sha256: 834b06e423d360c97197e7abec99b623fdc5ed3a0c39b88d6467e499074585e1
        - type: git
          url: https://github.com/foo/bar.git
          commit: 3f9389edc6cdf3f78a6896d550c236860aed62b2
          dest: 'src/deps/bar'

The contents of ``foo.tar.xz`` will be placed under
``.flatpak-builder/build/coolapp/`` and the contents of the ``bar.git``
repo will go under ``.flatpak-builder/build/coolapp/src/deps/bar``.

This is often useful to replace bundled dependencies inside sources
with custom versions.

Archive sources
---------------

These are sources of ``type: archive``. They are downloaded to the
download directory and extracted into the build directory when the build
starts.

A typical example of using this is:

.. code-block:: yaml

  modules:
    - name: foo
      buildsystem: meson
      sources:
        - type: archive
          url: https://example.org/foo-1.0.tar.xz
          mirror-urls:
            - https://mirror.example.com/foo-1.0.tar.xz
          sha256: 26c9007c655483438d83f17070190fc4b6bccb8ff5efc30fcedbb7d162681a7f
          dest-filename: foo.tar.xz
          strip-components: 2

- ``mirror-urls`` is a list of alternative urls that will be used for
  downloading, if the main ``url`` fails.

- The ``sha256`` is the sha256 checksum of the downloaded file, supplied
  by upstream of the source or calculated using ``sha256sum``.

- ``dest-filename`` is not usually used for this source. It is used to
  rename the downloaded file. This can be useful if the buildsystem
  expects a certain name for the archive.

- ``strip-components`` is the number of pathname components to strip
  during extraction. This defaults to ``1`` if not specified.

.. code-block::

  # orginal archive
  .
  └── a
      ├── b
      │   ├── c
      │   │   └── FILE
      │   └── FILE
      └── FILE

  # strip-components: 1
  .
  ├── b
  │   ├── c
  │   │   └── FILE
  │   └── FILE
  ├── FILE


  # strip-components: 2
  .
  ├── c
  │   └── FILE
  ├── FILE


``flatpak-builder`` supports
the following archive types: ``"rpm", "tar", "tar-gzip", "tar-compress", "tar-bzip2", "tar-lzip", "tar-lzma", "tar-lzop", "tar-xz", "tar-zst", "zip", "7z"``
The archive type is guessed from the suffix of the file basename.

``rpm`` is extracted with ``rpm2cpio``, ``zip`` is extracted with ``unzip``
and ``7z`` is extracted with ``7z``. The rest are extracted with
``tar``. These should be present on host so that flatpak-builder can
use them.The archive type is calculated from the suffix of the file
basename.

In case the correct archive type cannot be `guessed` from the filename
``archive-type: str`` can be used to manually specify the type where
``str`` is one of the above types.

Git sources
-----------

These are sources of ``type: git`` and it requires ``git`` to be
available on the host. A typical example of using this is:

.. code-block:: yaml

  modules:
    - name: foo
      buildsystem: meson
      sources:
        - type: git
          url: https://example.org/repo/foo.git
          tag: <git tag>
          commit: <commit hash>

- ``url`` is url of the git repository. The following schemes are
  usually used ``http://``, ``https://``, ``git://``, ``git+ssh://``,
  ``ssh://``.

.. tip::

  To use the ``file://`` scheme for a git repository, do
  ``git config --global protocol.file.allow always``.

- A ``branch: branch_name`` can also be used in place of a ``tag`` and
  ``commit``.

- ``commit`` is the commit to use from the git repository.

  In case of a tag, this must be the commit, the tag points to
  (``git rev-list -n 1 $tag``). In case of a ``branch``, it is verified
  that the branch points to this specific commit
  (``git show-ref "refs/heads/$branch"``).

.. tip::

  To ensure reproduciblity and avoid build failures, it is best to avoid
  using ``branch`` and always add a ``commit`` to a ``tag``.

By default ``flatpak-builder`` will do a shallow-clone of the git
repository and checkout submodules if any. ``disable-shallow-clone: true``
and ``disable-submodules`` can be used to override this behaviour.

``git`` source also supports using a local git repository with the
``path`` property.

.. note::

  Currently ``flatpak-builder`` does not support ``git lfs``. It can
  be worked around with using ``buildsystem: simple`` and running
  ``git lfs pull`` in the ``build-commands``.

File sources
------------

These are sources of ``type: file``. They are copied into the source
directory without any modifications.

Any arbitrary file type can be specified here along with
``buildsystem: simple`` to manually modify, build or install it
within ``build-commands``. This can be used in place of ``type: archive``
to manually extract an archive, uncompress squashfs filesystems
(like snaps) etc. provided the tool being used is made available inside
the sandbox.

A typical example is:

.. code-block:: yaml

  modules:
    - name: foo
      buildsystem: simple
      build-commands:
       - install -Dm0644 org.example.foo.metainfo.xml -t ${FLATPAK_DEST}/share/metainfo/
       - bsdtar -Oxf foo.deb 'data.tar.xz' | bsdtar -xf -
       - mv opt/ ${FLATPAK_DEST}/
       - ln -s ${FLATPAK_DEST}/opt/executable ${FLATPAK_DEST}/bin/executable
      sources:
        - type: file
          dest-filename: foo.deb
          url: https://example.org/foo-v1.0.deb
          sha256: <sha256 checksum of foo.deb>
        - type: file
          path: org.example.foo.metainfo.xml

    - name: unsquashfs
      buildsystem: simple
      build-commands:
       - XZ_SUPPORT=1 make -C squashfs-tools -j ${FLATPAK_BUILDER_N_JOBS} unsquashfs
       - install -Dpm755 -t "${FLATPAK_DEST}/bin" squashfs-tools/unsquashfs
      sources:
        - type: git
          url: https://github.com/plougher/squashfs-tools.git
          tag: 4.6.1
          commit: d8cb82d9840330f9344ec37b992595b5d7b44184

    - name: bar
      buildsystem: simple
      build-commands:
       - unsquashfs -dest bar -quiet -no-progress squashed.snap
       - mv bar ${FLATPAK_DEST}
       - ln -s ${FLATPAK_DEST}/bar/executable ${FLATPAK_DEST}/bin/executable
      sources:
        - type: file
          dest-filename: squashed.snap
          url: https://example.org/squashed-1.0.snap
          sha256: <sha256 checksum of squashed-1.0.snap>


``dest-filename`` is typically used with this source type to make the
filename `predictable` for the commands in the ``build-commands`` array.

Patch sources
-------------

These are sources of ``type: patch`` used with patchfiles. This requires
``patch`` and optionally ``git`` to be available on host. The typical
usage is to create patchfiles and place them somewhere relative to the
manifest root. The ``path`` property is the location of the file relative
to the manifest.

.. code-block:: yaml

  modules:
    - name: foo
      buildsystem: cmake-ninja
      builddir: true
      config-opts:
      - ...
    sources:
      - type: git
        url: https://example.org/foo.git
        tag: v1.0.0
        commit: 63c01f14391aef7d7691c7c63a610d47512147ed
      - type: patch
        path: patches/test-patch.patch

Multiple patches can be specified with ``paths``:

.. code-block:: yaml

  - type: patch
    paths:
      - patches/test1-patch.patch
      - patches/test2-patch.patch

By default, ``patch -p 1`` is used, but it can be adjusted with
``strip-components`` and additional options can be passed with
``options``.

The default patchfiles generated with ``git format-patch`` or ``git diff``
do not need any additional options.

.. code-block:: yaml

  - type: patch
    paths:
      - patches/test1-patch.patch
      - patches/test2-patch.patch
    options:
      - -d
      - dir1

``use-git: true`` and ``use-git-am: true`` can be used to use
``git apply`` and ``git am`` respectively instead of ``patch``.

Shell and Script sources
------------------------

A shell source with ``type: shell`` is used to modify a source during
extraction. ``commands`` takes in any shell commands.

This is usually used to make quick modifications to sources but a patch
should be used for any complex modification to make it fail safe.

.. code-block:: yaml

  modules:
    - name: foo
      sources:
        - type: archive
          url: https://example.org/foo.tar.xz
          sha256: ea029a2e21d2d6ad0a156f6679bd66836204aa78148a4c5e498fe682e77127ef
        - type: shell
          commands:
            - autoreconf -vfi

A script source with ``type: script`` is used to create self-contained
``/bin/sh`` scripts. ``commands`` takes in any shell commands and
``dest-filename`` can be used to specify the name for the resultant
file inside the source directory.

This is often useful if the executable requires custom arguments
to be passed before launch.

.. code-block:: yaml

  modules:
    - name: foo
      buildsystem: simple
      build-commands:
        - ...
        - install -Dm755 run ${FLATPAK_DEST}/bin/run
      sources:
        - type: archive
          url: https://example.org/foo-1.0.tar.xz
          sha256: 842ab8e69c94e985ba188cc22848c0515e9fa114380adcc08572d3fb9cfa19db
        - type: script
          dest-filename: run
          commands:
            - exec /app/foo/bin/foo_binary "$@"

Extra Data
-----------

Extra data is often used for proprietary applications to avoid
redistribution by downloading the binary package provided by the original
distributors during installation by the user. It may also be used for sources
with very large file sizes (over several gigabytes) to avoid storing a
huge amount of data in the ostree repository.

Since the package is downloaded only when installing, the application
metadata such as the desktop file, icon and the MetaInfo file must be
installed beforehand so that appstream can compose them to generate
catalogue data.

When installing the Flatpak package containing an extra data source,
Flatpak calls the ``apply_extra`` script which executes the command defined
in it. The dependencies needed to run the commands in the script must be
available in the sandbox.

The following properties must be present for an `extra-data` source:

- ``type: extra-data`` indicates that this is an extra-data source.
  Multiple such sources can exist.

- ``url`` should be a direct link to the binary source package. This is
  downloaded by Flatpak when installing the Flatpak package.

- ``sha256`` should be its SHA-256 checksum of the above package.

- ``size`` should be the file size of the package in `bytes` (``wc --bytes < file``).

- ``filename`` can be anything to name the downloaded source package.

The binary source packages are often specific to one architecture, so
``only-arches`` can be used to make the build specific to that
architecture only.

A typical extra-data manifest may look like:

.. code-block:: yaml

  modules:
    - shared-modules/lzo/lzo.json
    - shared-modules/squashfs-tools/squashfs-tools.json

    - name: dependency1
      ...

    - name: dependency2
      ...

    - name: foo
      buildsystem: simple
      build-commands:
        - install -Dm644 ${FLATPAK_ID}.metainfo.xml ${FLATPAK_DEST}/share/metainfo/${FLATPAK_ID}.metainfo.xml
        - install -Dm644 foo.desktop ${FLATPAK_DEST}/share/applications/${FLATPAK_ID}.desktop
        - install -Dm644 foo.svg ${FLATPAK_DEST}/share/icons/hicolor/scalable/apps/${FLATPAK_ID}.svg
        - ln -s ${FLATPAK_DEST}/extra/subdir/foo_binary ${FLATPAK_DEST}/bin/foo_binary
        - install -Dm755 apply_extra ${FLATPAK_DEST}/bin/apply_extra
      sources:
        - type: extra-data
          filename: foo.snap
          only-arches:
            - x86_64
          url: https://example.org/foo_v1.2.snap
          sha256: 842ab8e69c94e985ba188cc22848c0515e9fa114380adcc08572d3fb9cfa19db
          size: 123101184
        - type: file
          path: com.flatpak.foo.metainfo.xml
        - type: file
          path: foo.desktop
        - type: file
          path: foo.svg
        - type: script
          dest-filename: apply_extra
          commands:
            - unsquashfs -quiet -no-progress foo.snap
            - mv squashfs-root/usr/lib/foo/* .
            - rm -r squashfs-root foo.snap

The ``buildsystem`` is ``simple`` as the module is manually installed.

The ``apply_extra`` script here requires
``unsquashfs`` to run, so that is built and installed.

.. note::

  Flatpak supports exporting icons, desktop files etc. from
  ``/app/extra/export/share/`` however when creating a Flatpak package
  for a software store like Flathub or when composing the metadata with
  appstreamcli, desktop files and icons must not be placed in
  ``/app/extra/export/`` and instead should go to the proper locations
  documented in :doc:`conventions`.

The icon, desktop file and Metainfo must be added as sources. The
``path`` indicates that they reside in the same directory relative to the
manifest. The first three lines of the ``build-commands`` installs them
so that ``appstream-compose`` can compose the metadata.

The last line creates an empty symlink from ``${FLATPAK_DEST}/extra/``
to ``${FLATPAK_DEST}/bin/`` so that the executable is found in ``$PATH``
inside the sandbox and can be used in the top-level ``command: foo_binary``
property. Instead of a symlink this also often a script like:

.. code-block:: shell

  #!/bin/sh

  exec /app/extra/subdir/foo_binary "$@"


The ``subdir`` directory comes from the contents of the extracted snap
and how that is installed.

.. note::

  Note that variables like ``$FLATPAK_DEST`` are not available in the
  runtime sandbox or in the sandbox where ``apply_extra`` is executed
  when installing the Flatpak.

  Please avoid using them when creating the script in the manifest
  as this will be incorrectly expanded.

The commands needed to extract the snap are specified in the ``apply_extra``
script. These can be any shell commands that run when installing the
Flatpak package but note that it won't have access to anything outside
``/app/extra``. The script is run in a sandbox
(equivalent to ``flatpak run --sandbox``) and won't have network, dbus or
any host access. The ``apply_extra`` can itself be a seperate script file
too but must be installed as ``${FLATPAK_DEST}/bin/apply_extra``.

``unsquashfs -quiet -no-progress foo.snap`` extracts the snap package
into a directory called ``squashfs-root``, then the contents of that
direcory are moved to ``/app/extra`` and lastly ``squashfs-root`` and the
snap package itself is cleaned up to save space.

This is another example that can be used to extract Debian packages (``.deb``).

.. code-block:: yaml

  - name: foo
    buildsystem: simple
    build-commands:
      - install -Dm755 apply_extra ${FLATPAK_DEST}/bin/apply_extra
      - install -Dm644 -t ${FLATPAK_DEST}/share/metainfo/ ${FLATPAK_ID}.metainfo.xml
      - install -Dm644 -t ${FLATPAK_DEST}/share/applications/ ${FLATPAK_ID}.desktop
      - install -Dm644 -t ${FLATPAK_DEST}/share/icons/hicolor/scalable/apps/ ${FLATPAK_ID}.svg
      - ln -s ${FLATPAK_DEST}/extra/bin/foo_binary ${FLATPAK_DEST}/bin/foo_binary

    sources:
      - type: script
        dest-filename: apply_extra
        commands:
          - bsdtar --to-stdout -xf foo.deb data.* | bsdtar -xf -
          - rm foo.deb

Flatpak extensions can also use extra-data sources in a similar manner.
This is an example of an extension manifest utilising an extra-data
source.

.. code-block:: yaml

  app-id: com.foo.app.Plugins.audio
  runtime: com.foo.app
  sdk: org.freedesktop.Sdk//25.08
  build-extension: true
  separate-locales: false
  modules:
    - name: foo
      buildsystem: simple
      build-commands:
        - install -Dm755 apply_extra -t $FLATPAK_DEST/bin/
      sources:
        - type: script
          dest-filename: apply_extra
          commands:
            - bsdtar -xf foo.qdz --strip-components=3
            - rm -f foo.qdz
        - type: extra-data
          filename: foo.qdz
          url: http://example.org/foo_v1.qdz
          sha256: 2027ed7e5935612a38c3f7b751b6bd23cfb5b06d1a1e5f34907bc021b09de225
          size: 523360406

For more information on Flatpak extensions please see :doc:`extension`.

Directory source
----------------

A directory source with ``type: dir`` is best suited for use in CI to
build the latest changes and avoid downloading multiple times. These
don't support any caching, so it will be rebuilt each time the
application is being built.

When submitting an application to software stores like Flathub, ``dir``
should be avoided altogether.

``path`` should be the path of the local directory relative to the
manifest root path, whose contents will be copied during build.

.. code-block:: yaml

  modules:
    - name: foo
      buildsystem: simple
      build-commands:
        - ...
      sources:
        - type: dir
          path: icons

Additonally there are `bzr`, `svn` and `inline` sources supported.
`bzr` and `svn` requires the `bzr <https://code.launchpad.net/bzr>`_
and `svn <https://subversion.apache.org/>`_ commandline tools to be
installed respectively. Please see
:doc:`flatpak-builder-command-reference` for them.
