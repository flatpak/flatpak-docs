Multiarch support
=================

Flatpak has multiarch/multilib support, but it's not enabled by default and
require some additional steps to enable it. This section covers enabling
multiarch/multilib in your application bundle.

Running 32-bit programs
-----------------------

In order to set up the run time environment for 32-bit executables, first you'll
need to allow it in ``finish-args``:

.. code-block:: yaml

  finish-args:
    - --allow=multiarch

This is enough for static binaries, but most real-world GNU/Linux programs are
linked dynamically. Those need some shared libraries to work.

Freedesktop.org and GNOME SDKs both provide a special flatpak extension with a
set of libraries for corresponding architecture. This extension can be attached
to an app of different architecture. In order to enable the extension for your
app, define an extension point for it in the app manifest:

.. code-block:: yaml

  add-extensions:
    org.freedesktop.Platform.Compat.i386:
      directory: lib/i386-linux-gnu
      version: '20.08'

    # This is not strictly required, but needed for debugging 32-bit programs
    org.freedesktop.Platform.Compat.i386.Debug:
      directory: lib/debug/lib/i386-linux-gnu
      version: '20.08'
      no-autodownload: true

For GNOME runtime, use ``org.gnome.Platform.Compat.i386`` instead.

Note that this extension ``version`` must match the ``runtime-version`` of the 
application.

If the 32-bit programs make use of GPU acceleration, or have some graphical UI
in general, you'll also need 32-bit GL drivers. Add an extension point for it:

.. code-block:: yaml

  add-extensions:
    org.freedesktop.Platform.GL32:
      directory: lib/i386-linux-gnu/GL
      version: '1.4'
      versions: 20.08;1.4
      subdirectories: true
      no-autodownload: true
      autodelete: false
      add-ld-path: lib
      merge-dirs: vulkan/icd.d;glvnd/egl_vendor.d;OpenCL/vendors;lib/dri;lib/d3d;vulkan/explicit_layer.d;vulkan/implicit_layer.d
      download-if: active-gl-driver
      enable-if: active-gl-driver

Note that the ``versions`` property here must contain both ``1.4`` and the same
value as in ``runtime-version``.

Make sure to create directories where the extensions will be mounted (the mount
points are specified in ``directory`` properties and are relative to the app
bundle mount point, i.e. to ``/app/``). This can be done at stage of the build.

Finally, you need to make the dynamic library loader know the paths to 32-bit
libraries. In order to do this, you can install a ``/app/etc/ld.so.conf`` file
with contents like this:

.. code-block:: text

  /app/lib32
  /app/lib/i386-linux-gnu

Here ``/app/lib32`` is the directory where you install additional 32-bit
libraries, if any.

You can combine the above two steps in a special module, e.g.

.. code-block:: yaml

  modules:
    - name: bundle-setup
      buildsystem: simple
      build-commands:
        - mkdir -p /app/lib/i386-linux-gnu
        - mkdir -p /app/lib/debug/lib/i386-linux-gnu
        - mkdir -p /app/lib/i386-linux-gnu/GL
        - install -Dm644 ld.so.conf /app/etc/ld.so.conf
      sources:
        - type: file
          dest-filename: ld.so.conf
          url: data:/app/lib32%0A/app/lib/i386-linux-gnu%0A
