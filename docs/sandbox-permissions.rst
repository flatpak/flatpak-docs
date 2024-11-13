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

.. note::

  Note, that these permissions are completely static and variable
  expansion or substitution (for example in ``--filesystem`` or ``--env``)
  is not possible.

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
commonly require, and can therefore be freely used.

- ``--allow=bluetooth`` - Allow access to Bluetooth (``AF_BLUETOOTH``) sockets
- ``--device=dri`` - OpenGL rendering
- ``--share=ipc`` - Share IPC namespace with the host [#f1]_
- ``--share=network`` - Access the network [#f2]_
- ``--socket=cups`` - Talk to the CUPS printing system (``$CUPS_SERVER`` or server defined in CUPS's ``client.conf``. Falls back to ``/var/run/cups/cups.sock``)
- ``--socket=gpg-agent`` - Talk to the GPG agent (The socket in ``gpgconf --list-dir agent-socket``)
- ``--socket=pcsc`` - Smart card access ``$PCSCLITE_CSOCK_NAME``
- ``--socket=pulseaudio`` - Access to PulseAudio, includes sound input (mic), sound output/playback, MIDI and ALSA sound devices in ``/dev/snd``
- ``--socket=ssh-auth``- Allow access to ``$SSH_AUTH_SOCK``

.. note::

  Applications that do not support native Wayland should use
  only ``--socket=x11`` and applications that do, should use
  ``--socket=fallback-x11`` and ``--socket=wayland``.
  The two configurations here will make the application work on both
  X11 and Wayland sessions of the desktop environment.

- ``--socket=wayland`` - Show windows with Wayland
- ``--socket=x11`` - Show windows using X11
- ``--socket=fallback-x11`` - Show windows using X11, if Wayland is not
  available, overrides ``x11`` socket permission. Note that you must
  still use ``--socket=wayland`` for wayland permission

D-Bus access
````````````

DBus access is filtered by default. The default policy for the session bus
only allows the application to own its own namespace named by
``$FLATPAK_ID``, subnames of it and ``org.mpris.MediaPlayer2.$FLATPAK_ID``
for `MPRIS <https://www.freedesktop.org/wiki/Specifications/mpris-spec/>`_.
Furthermore, it is only allowed to talk to names matching those patterns,
the bus itself ``org.freedesktop.DBus`` and portal APIs of the form
``org.freedesktop.portal.*``.

Access to the entire bus with ``--socket=system-bus`` or
``--socket=session-bus`` stops the filtering and using them is a security
risk. So they must be avoided, unless the application is a development
tool.

``flatpak run --log-session-bus $FLATPAK_ID`` can be used to find the specific
D-Bus permissions needed.

**Ownership**

Any ownership beyond what is granted by default ie. own namespace and
``org.mpris.MediaPlayer2.$FLAPTAK_ID`` is typically unnecessary
although there can be exceptions.

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
- Using :ref:`conventions:XDG base directories` to store application's
  cache, config and state. Then no additional filesystem access would be
  required.
- Avoiding full home access and instead using XDG directories such
  as ``xdg-music`` or ``xdg-download`` etc.

The following permission options are available:

- ``:ro`` - read-only access
- ``:create`` - read/write access, and create the directory if it doesn't
  exist

Additionally the following permissions are available:

====================  ==============================================================================================================================  ===================================================
``host``              Access to ``/home, /media, /opt, /run/media, /srv`` and everything provided by ``host-os, host-etc`` mounted in ``/run/host``    Includes any subpaths
``host-etc``          Host's ``/etc``                                                                                                                  Host's ``/etc`` is mounted at ``/run/host/etc``
``host-os``           Host's ``/usr, /bin, /sbin, /lib{32, 64}, /etc/ld.so.cache, /etc/alternatives``                                                  Mounted at ``/run/host``
``home``              Access the home directory                                                                                                        Except ``~/.var/app``
``/some/dir``         Access an arbitrary path except any reserved path                                                                                Includes any subpaths
``~/some/dir``        Arbitrary path relative to the home directory                                                                                    Includes any subpaths
``xdg-desktop``       Access the XDG desktop directory                                                                                                 ``$XDG_DESKTOP_DIR`` or ``$HOME/Desktop``
``xdg-documents``     Access the XDG documents directory                                                                                               ``$XDG_DOCUMENTS_DIR`` or ``$HOME/Documents``
``xdg-download``      Access the XDG download directory                                                                                                ``$XDG_DOWNLOAD_DIR`` or ``$HOME/Downloads``
``xdg-music``         Access the XDG music directory                                                                                                   ``$XDG_MUSIC_DIR`` or ``$HOME/Music``
``xdg-pictures``      Access the XDG pictures directory                                                                                                ``$XDG_PICTURES_DIR`` or ``$HOME/Pictures``
``xdg-public-share``  Access the XDG public directory                                                                                                  ``$XDG_PUBLICSHARE_DIR`` or ``$HOME/Public``
``xdg-videos``        Access the XDG videos directory                                                                                                  ``$XDG_VIDEOS_DIR`` or ``$HOME/Videos``
``xdg-templates``     Access the XDG templates directory                                                                                               ``$XDG_TEMPLATES_DIR`` or ``$HOME/Templates``
``xdg-config``        Access the XDG config directory [#f3]_                                                                                           ``$XDG_CONFIG_HOME`` or ``$HOME/.config``
``xdg-cache``         Access the XDG cache directory  [#f3]_                                                                                           ``$XDG_CACHE_HOME`` or ``$HOME/.cache``
``xdg-data``          Access the XDG data directory   [#f3]_                                                                                           ``$XDG_DATA_HOME`` or ``$HOME/.local/share``
``xdg-run/path``      Access subdirectories of the XDG runtime directory                                                                               ``$XDG_RUNTIME_DIR/path`` (``/run/user/$UID/path``)
====================  ==============================================================================================================================  ===================================================

Except ``host, host-etc, host-os`` paths can be added to all the above
filesystem options. For example, ``--filesystem=xdg-documents/path``.

Other filesystem access guidelines include:

- The ``--persist=DIR`` option can be used to map directories from the
  user's home directory into the sandbox filesystem. This only works if
  the application has no ``home`` or a broader permission like ``host``
  that includes ``home``.

  For example, if an application hardcodes the directory ``~/.foo``,
  without any ``home`` access and no ``--persist`` the directory will be
  lost from the sandbox once exited due to the filesystem being set up
  as tmpfs by flatpak unless overriden. A ``--persist=.foo`` bind mounts
  ``~/.foo`` `inside the sandbox` to ``~/.var/app/$FLATPAK_ID/.foo`` on
  host thus allowing an app to persistently store data in
  ``~/.var/app/$FLATPAK_ID/.foo`` which would otherwise be lost.

  A ``--persist=.`` will `persist` all directories.

  This does not support ``:create, :ro, :rw`` suffixes or
  special values like ``xdg-documents``. However, the directory will be
  created by flatpak if it doesn't already exist.

  This makes it possible to avoid configuring access to the entire home
  directory, and can be useful for applications that hardcode file paths
  in ``~/``.
- If an application uses ``$TMPDIR`` to contain lock files you may want to
  add a wrapper script that sets it to
  ``$XDG_RUNTIME_DIR/app/$FLATPAK_ID`` (tmpfs) or ``/var/tmp`` (persistent
  on host).
- Retaining and sharing configuration with non-Flatpak installations is to
  be avoided.

Reserved Paths
``````````````

The following paths and subpaths of them are reserved and asking access
to them with ``--filesystem`` will have no effect::

/app, /bin, /dev, /etc, /lib, /lib32, /lib64, /proc, /run/flatpak, /run/host, /sbin, /usr

The entire ``/run`` is not allowed and all subpaths of ``/run`` except
``/run/flatpak, /run/host`` is allowed to be exposed via
``--filesystem``.

Additionally the following directories from host need to be explicitly
requested with ``--filesystem`` and are not available with
``home, host, host-os, host-etc`` by default:

- ``~/.var/app`` - The app can access only its own directory in ``~/.var/app/$FLATPAK_ID``
- ``$XDG_DATA_HOME/flatpak`` (``~/.local/share/flatpak``)
- ``/boot``
- ``/efi``
- ``/root``
- ``/sys``
- ``/tmp``
- ``/var`` - Note that by default ``/var/{cache, config, data, tmp}``
  inside the sandbox are the same as ``~/.var/app/$FLATPAK_ID/{cache, config, data, cache/tmp}``.
  However an explicit ``--filesystem=/var`` will make only ``/var`` from
  host available and those will no longer be available.
- ``/var/lib/flatpak`` - ``/var`` does not give access to this.

Device access
`````````````
You can provide the following device permissions:

========= ======================================================
``dri``   Direct Rendering Interface. Necessary for GL.
``kvm``   Kernel based Virtual Machine ``/dev/kvm``
``shm``   Shared Memory in ``/dev/shm``.
``input`` Input devices as exposed in ``/dev/input``. This includes game controllers. Since Flatpak 1.15.6.
``usb``   Raw USB devices as exposed in ``/dev/bus/usb``. Since Flatpak 1.15.11.
``all``   All devices, including all of the above except ``shm``
========= ======================================================

.. note::

  Using newer permissions like ``input`` or ``usb`` will have no effect
  on older Flatpak versions and will fail when used through Flatpak
  commandline.

While not ideal, ``--device=all`` can be used to access devices like
webcams, CD/DVD drives etc.

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
.. [#f3] ``xdg-{cache, config, data}`` bind mounts the paths from host to the per-app sandbox directory.
   Inside the sandbox ``$XDG_CACHE_HOME``, ``$XDG_CONFIG_HOME`` and ``$XDG_DATA_HOME`` is set to
   ``$HOME/.var/app/$FLATPAK_ID/{cache, config, data}`` respectively. So for example, ``xdg-data/applications`` ie.
   ``$XDG_DATA_HOME/applications`` on host is bind mounted to ``$HOME/.var/app/$FLATPAK_ID/data/applications``
   (inside the sandbox this is ``$XDG_DATA_HOME/applications``).
   Additionally it'll have two mount points - one expanded to
   ``$XDG_DATA_HOME/applications`` from the host and another to the
   sandbox's ``$XDG_DATA_HOME/applications`` ie. ``$HOME/.var/app/$FLATPAK_ID/data/applications``.
