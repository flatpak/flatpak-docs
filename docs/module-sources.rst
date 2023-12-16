Module Sources
==============

A list of available module sources can be found in :doc:`manifests`. This 
guide will go in depth on each of them and provide examples.

Sources are copied or downloaded to the source directory before the build
starts.

The ``type`` property of a source determines how that source will be
handled before or during build by ``flatpak`` or ``flatpak-builder``.

Most of the sources support ``path`` and ``url`` property where ``path``
is relative to the root of manifest and ``url`` is a direct url to the
source. 

The ``url`` property requires a ``checksum`` of the file to be specified 
after it using ``sha256`` or ``sha512``.

Archive sources
---------------

These are sources of ``type: archive``. They are downloaded to the 
download directory and extracted into the build directory when the build 
starts.

The download directoy is ``.flatpak-builder/downloads/<checksum of source>/``.

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

.. code-block:: plain

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
They type of the archive is `gussed` from the suffix of the file basename.

``rpm`` is extracted with ``rpm2cpio``, ``zip`` is extracted with ``unzip``
and ``7z`` is extratced with ``7z``. The rest are extracted with
``tar``.

In case the correct archive type cannot be `guessed` from the filename
``archive-type: str`` can be used to manually specify the type where
``str`` is one of the above types.

Git sources
-----------

These are sources of ``type: git``. A typical example of using this is:

.. code-block:: yaml

  modules:
    - name: foo
      buildsystem: meson
      sources:
        - type: git
          url: https://example.org/repo/foo.git
          tag: <git tag>
          commit: <commit hash>>

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

There are a few properties common to all source types ``only-arches``,
``skip-arches`` and ``dest``.

The ``only-arches`` property is used to indicate 
