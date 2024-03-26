Sandbox Permissions
===================

One of Flatpak's main goals is to increase the security of desktop systems by
isolating applications from one another. This is achieved using sandboxing
and means that, by default, applications that are run with Flatpak have
extremely limited access to the host environment. This includes:

- No access to any host files except the runtime, the app,
  ``~/.var/app/$FLATPAK_ID``, and ``$XDG_RUNTIME_DIR/app/$FLATPAK_ID``.
  Only the latter two being writable.
- No access to the network.
- No access to any device nodes (apart from ``/dev/null``, etc).
- No access to processes outside the sandbox.
- Limited syscalls.  For instance, apps can't use nonstandard network socket
  types or ptrace other processes.
- Limited access to the session D-Bus instance - an app can only own its
  own name on the bus.
- No access to host services like X11, system D-Bus, or PulseAudio.

Most applications will need access to some of these resources in order to
be useful. This is primarily done during the finishing build stage, which
can be configured through the ``finish-args`` section of the manifest file
(see :doc:`manifests`).

Portals
-------

Portals have already been mentioned in :doc:`basic-concepts`. They are a
framework for providing access to resources outside of the sandbox, including:

- Opening files with a native file chooser dialog
- Opening URIs
- Printing
- Showing notifications
- Taking screenshots
- Inhibiting the user session from ending, suspending, idling or getting
  switched away
- Getting network status information

In many cases, portals use a system component to implicitly ask the user
for permission before granting access to a particular resource. For example,
in the case of opening a file, the user's selection of a file using the file
chooser dialog is interpreted as implicitly granting the application access
to whatever file is chosen.

This approach enables applications to avoid having to configure blanket
access to large amounts of data or services and gives users control over
what their applications have access to.

Interface toolkits like GTK3 and Qt5 implement transparent support for
portals, meaning that applications don't need to do any additional
work to use them (it is worth checking which portals each toolkit
supports). Applications that aren't using a toolkit with support
for portals can refer to the `xdg-desktop-portal API documentation
<https://flatpak.github.io/xdg-desktop-portal/>`_ for
information on how to use them.

Permissions guidelines
----------------------

While application developers have control over the sandbox permissions they
wish to configure, good practice is encouraged and can be enforced. For
example, the Flathub hosting service places requirements on which permissions
can be used, and software on the host may warn users if certain permissions
are used.

The following guidelines describe which permissions can be freely used,
which can be used on an as-needed basis, and which should be avoided.

Standard permissions
````````````````````

The following permissions provide access to basic resources that applications
commonly require, and can therefore be freely used:

- ``--allow=bluetooth`` - Allow access to Bluetooth
- ``--device=dri`` - OpenGL rendering
- ``--device=input`` - Access to ``/dev/input``
- ``--share=ipc`` - share IPC namespace with the host [#f1]_
- ``--share=network`` - access the network [#f2]_
- ``--socket=cups`` - Talk to the CUPS printing system
- ``--socket=gpg-agent`` - Talk to the GPG agent
- ``--socket=pcsc`` - Smart card access
- ``--socket=pulseaudio`` - Play sound with PulseAudio
- ``--socket=ssh-auth``- Allow access to ``SSH_AUTH_SOCK``
- ``--socket=wayland`` - Show windows with Wayland
- ``--socket=x11`` - show windows using X11
- ``--socket=fallback-x11`` - show windows using X11, if Wayland is not
  available, overrides ``x11`` socket permission. Note that you must
  still use ``--socket=wayland`` for wayland permission

D-Bus access
````````````

Access to the entire bus with ``--socket=system-bus`` or
``--socket=session-bus`` is a security risk and must be avoided, unless
the application is a development tool.

``flatpak run --log-session-bus <appid>`` can be used to find the specific
D-Bus permissions needed.

**Ownership**

Applications are automatically granted access to their own namespace. Ownership
beyond this is typically unnecessary, although there are a small
number of exceptions, such as using `MPRIS to provide media controls
<https://www.freedesktop.org/wiki/Specifications/mpris-spec/>`_.

**Talk**

Talk permissions can be freely used, although it is recommended to use the
minimum required.

Filesystem access
`````````````````

As a general rule, static and permanent filesystem access should be
limited as much as possible. This includes:

- Using portals as an alternative to blanket filesystem access, wherever
  possible.
- Using read-only access wherever possible, using the ``:ro`` option.
- If some home directory access is absolutely required, using XDG directory
  access only.

The following permission options are available:

- ``:ro`` - read-only access
- ``:create`` - read/write access, and create the directory if it doesn't
  exist

Additionally the following permissions are available:

====================  ===============================================================  ===================================================
``host``              Access all files [#f3]_                                           
``host-etc``          Access all files in host and host's /etc [#f3]_
``home``              Access the home directory [#f4]_
``/some/dir``         Access an arbitrary path [#f5]_ [#f6]_
``~/some/dir``        Access an arbitrary path relative to the home directory [#f6]_
``xdg-desktop``       Access the XDG desktop directory                                  ``$XDG_DESKTOP_DIR`` or ``$HOME/Desktop``
``xdg-documents``     Access the XDG documents directory                                ``$XDG_DOCUMENTS_DIR`` or ``$HOME/Documents``
``xdg-download``      Access the XDG download directory                                 ``$XDG_DOWNLOAD_DIR`` or ``$HOME/Downloads``
``xdg-music``         Access the XDG music directory                                    ``$XDG_MUSIC_DIR`` or ``$HOME/Music``
``xdg-pictures``      Access the XDG pictures directory                                 ``$XDG_PICTURES_DIR`` or ``$HOME/Pictures``
``xdg-public-share``  Access the XDG public directory                                   ``$XDG_PUBLICSHARE_DIR`` or ``$HOME/Public``
``xdg-videos``        Access the XDG videos directory                                   ``$XDG_VIDEOS_DIR`` or ``$HOME/Videos``
``xdg-templates``     Access the XDG templates directory                                ``$XDG_TEMPLATES_DIR`` or ``$HOME/Templates``
``xdg-config``        Access the XDG config directory [#f7]_                            ``$XDG_CONFIG_HOME`` or ``$HOME/.config``
``xdg-cache``         Access the XDG cache directory  [#f7]_                            ``$XDG_CACHE_HOME`` or ``$HOME/.cache``
``xdg-data``          Access the XDG data directory   [#f7]_                            ``$XDG_DATA_HOME`` or ``$HOME/.local/share``
``xdg-run/path``      Access subdirectories of the XDG runtime directory                ``$XDG_RUNTIME_DIR/path`` (``/run/user/$UID/path``)
====================  ===============================================================  ===================================================

Note that ``host, host-etc, host-os`` mounts the host directories under
``/run/host`` inside the sandbox to avoid conflict with the runtime.

Except ``host, host-etc, host-os`` paths can be added to all the above
filesystem options. For example, ``--filesystem=xdg-documents/path``.

Other filesystem access guidelines include:

- The ``--persist=path`` option can be used to map paths from the user's
  home directory into the sandbox filesystem.
  This makes it possible to avoid configuring access to the entire home
  directory, and can be useful for applications that hardcode file paths in
  ``~/``.
- If an application uses ``$TMPDIR`` to contain lock files you may want to
  add a wrapper script that sets it to ``$XDG_RUNTIME_DIR/app/$FLATPAK_ID``.
- Retaining and sharing configuration with non-Flatpak installations is to
  be avoided.

Device access
`````````````

While not ideal, ``--device=all`` can be used to access devices like
controllers or webcams.

dconf access
````````````

As of xdg-desktop-portal 1.1.0 and glib 2.60.5 (in the runtime) you do not
need direct DConf access in most cases.

As of now this glib version is included in ``org.freedesktop.Platform//19.08``
and ``org.gnome.Platform//3.34`` and newer.

If an application existed prior to these runtimes you can tell Flatpak (>=
1.3.4) to migrate the DConf settings on the
host into the sandbox by adding
``--metadata=X-DConf=migrate-path=/org/example/foo/`` to ``finish-args``. The
path must be similar to your app-id or it will not be allowed (case is
ignored and ``_`` and ``-`` are treated equal).

If you are targeting older runtimes or require direct DConf access for other
reasons you can use these permissions::

  --filesystem=xdg-run/dconf
  --filesystem=~/.config/dconf:ro
  --talk-name=ca.desrt.dconf
  --env=DCONF_USER_CONFIG_DIR=.config/dconf

With those permissions glib will continue using dconf directly.

If you use a newer runtime where dconf is no longer built and still need it
you will have to build the `dconf <https://download.gnome.org/sources/dconf/>`_ GIO module
and set ``--env=GIO_EXTRA_MODULES=/app/lib/gio/modules/``.

gvfs access
```````````

As of gvfs 1.48, the gvfs daemons and applications use an on-disk socket
to communicate, rather than an abstract socket so that the gvfs infrastructure
still works when network support is disabled in the application's sandbox.

A number of different options need to be passed depending on the application's
use of gvfs.

``--talk-name=org.gtk.vfs.*`` is necessary to talk to the gvfs daemons over
D-Bus and list mounts using the GIO APIs.

``--filesystem=xdg-run/gvfsd`` is necessary to use the GIO APIs to list and access
non-native files using the GIO APIs, using URLs rather than FUSE paths.

``--filesystem=xdg-run/gvfs`` is necessary to give access to the FUSE mounts
non-GIO and legacy applications can use. This is what will make native files
appear under ``/run/user/`id -u`/gvfs/``.

Typical GNOME and GTK applications should use::

  --talk-name=org.gtk.vfs.*
  --filesystem=xdg-run/gvfsd

Typical non-GNOME and non-GTK applications should use::

  --filesystem=xdg-run/gvfs

No application should be using ``--talk-name=org.gtk.vfs`` in its manifest, as
there are no D-Bus services named ``org.gtk.vfs``.

External drive access
`````````````````````

External drives are mounted by the host system using systemd, udev, udisk
fstab etc. and each of them can have different defaults. Flatpak has no
control over how and where they get mounted. The following
filesystem permissions should work in most cases::

  --filesystem=/media
  --filesystem=/run/media
  --filesystem=/mnt

If ``--filesystem=host`` is used ``/media, /run/media`` is shared
automatically if they exist.

Note that these should not have subpaths in them unless the value
of the subpath can be consistently pre-determined. Block device naming
depends on the kernel/fstab configuration and cannot be pre-determined.

.. rubric:: Footnotes

.. [#f1] This is not necessarily required, but without it the X11 shared
   memory extension will not work, which is very bad for X11 performance.
.. [#f2] Giving network access also grants access to all host services
   listening on abstract Unix sockets (due to how network namespaces work),
   and these have no permission checks. This unfortunately affects e.g. the X
   server and the session bus which listens to abstract sockets by default. A
   secure distribution should disable these and just use regular sockets.
.. [#f3] Except for ``/app, /bin, /boot, /efi, /etc, /lib, /lib32, /lib64, /proc, /root, /run, /sbin, /tmp, /usr, /var``
.. [#f4] This does not include access to folders under ``~/.var/app`` except the application's own
.. [#f5] Except ``/app, /dev, /etc, /lib, /lib32, /lib64, /proc, /root, /run/flatpak, /run/host, /sbin, /usr``
.. [#f6] The arbitrary path includes all its subfolders and subfiles if any.
.. [#f7] ``xdg-{cache, config, data}`` binds mount the paths from host to the per-app sandbox directory.
   Inside the sandbox ``$XDG_CACHE_HOME, $XDG_CONFIG_HOME and $XDG_DATA_HOME`` is set to
   ``$HOME/.var/app/<app-id>/{cache, config, data}``. So this permission is not needed
   unless access to the host directory, bind mounted to
   ``$HOME/.var/app/<app-id>/{cache, config, data}`` is desired.
