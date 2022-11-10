Sandbox Permissions Reference
=============================

Sandbox permissions can be configured from an application manifest file
(see :doc:`manifests`). They can also be set with the ``build-finish``,
``run`` and ``override`` commands.

The following list includes many of the most useful permission options. A
complete list can be viewed using ``flatpak build-finish --help``.

===================================================  ===========================================
``--socket=x11``                                     Show windows using X11
``--socket=fallback-x11``                            Grant X11 access when Wayland is not available
``--share=ipc``                                      Share IPC namespace with the host [#f1]_
``--allow=bluetooth``                                Allow access to Bluetooth
``--device=dri``                                     OpenGL rendering
``--socket=wayland``                                 Show windows using Wayland
``--socket=pulseaudio``                              Play sounds using PulseAudio
``--share=network``                                  Access the network [#f2]_
``--talk-name=org.freedesktop.secrets``              Talk to a named service on the session bus
``--system-talk-name=org.freedesktop.GeoClue2``      Talk to a named service on the system bus
``--socket=cups``                                    Talk to the CUPS printing system
``--socket=gpg-agent``                               Talk to the GPG agent
``--socket=pcsc``                                    Grant access to smart card
``--socket=ssh-auth``                                SSH authentication
``--socket=session-bus``                             Unlimited access to user's D-Bus session
``--socket=system-bus``                              Unlimited access to all of D-Bus
===================================================  ===========================================

Filesystem permissions
----------------------

Each of the following permissions configure filesystem access, and should
be added to ``--filesystem=``:

====================  ===========================================
``host``              Access all files [#f3]_
``home``              Access the home directory
``/some/dir``         Access an arbitrary path
``~/some/dir``        Access an arbitrary path relative to the home directory
``xdg-desktop``       Access the XDG desktop directory
``xdg-documents``     Access the XDG documents directory
``xdg-download``      Access the XDG download directory
``xdg-music``         Access the XDG music directory
``xdg-pictures``      Access the XDG pictures directory
``xdg-public-share``  Access the XDG public directory
``xdg-videos``        Access the XDG videos directory
``xdg-templates``     Access the XDG templates directory
``xdg-config``        Access the XDG config directory
``xdg-cache``         Access the XDG cache directory
``xdg-data``          Access the XDG data directory
``xdg-run/path``      Access subdirectories of the XDG runtime directory (where path is any subdirectory)
====================  ===========================================

Paths can be added to all the above filesystem options. For example,
``--filesystem=xdg-documents/path``. The following permission options can
also be added:

- ``:ro`` - read-only access
- ``:rw`` - read/write access (this is the default)
- ``:create`` - read/write access, and create the directory if it doesn't exist

.. rubric:: Footnotes

.. [#f1] This is not necessarily required, but without it the X11 shared
   memory extension will not work, which is very bad for X11 performance.
.. [#f2] Giving network access also grants access to all host services
   listening on abstract Unix sockets (due to how network namespaces work),
   and these have no permission checks. This unfortunately affects e.g. the X
   server and the session bus which listens to abstract sockets by default. A
   secure distribution should disable these and just use regular sockets.
.. [#f3] Except for the blacklisted paths mentioned in :doc:`sandbox-permissions`.

