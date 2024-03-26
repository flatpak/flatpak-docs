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

====================  ========================================================  ===================================================
``host``              Access all files [#f3]_
``host-etc``          Access all files in /etc
``host-os``           Access to host operating system's libraries, executables  Mounted in ``/run/host`` inside the sandbox
``home``              Access the home directory
``/some/dir``         Access an arbitrary path [#f4]_ [#f5]_
``~/some/dir``        Access an arbitrary path relative to the home directory
                      [#f5]_
``xdg-desktop``       Access the XDG desktop directory                          ``$XDG_DESKTOP_DIR`` or ``$HOME/Desktop``
``xdg-documents``     Access the XDG documents directory                        ``$XDG_DOCUMENTS_DIR`` or ``$HOME/Documents``
``xdg-download``      Access the XDG download directory                         ``$XDG_DOWNLOAD_DIR`` or ``$HOME/Downloads``
``xdg-music``         Access the XDG music directory                            ``$XDG_MUSIC_DIR`` or ``$HOME/Music``
``xdg-pictures``      Access the XDG pictures directory                         ``$XDG_PICTURES_DIR`` or ``$HOME/Pictures``
``xdg-public-share``  Access the XDG public directory                           ``$XDG_PUBLICSHARE_DIR`` or ``$HOME/Public``
``xdg-videos``        Access the XDG videos directory                           ``$XDG_VIDEOS_DIR`` or ``$HOME/Videos``
``xdg-templates``     Access the XDG templates directory                        ``$XDG_TEMPLATES_DIR`` or ``$HOME/Templates``
``xdg-config``        Access the XDG config directory [#f6]_                    ``$XDG_CONFIG_HOME`` or ``$HOME/.config``
``xdg-cache``         Access the XDG cache directory  [#f6]_                    ``$XDG_CACHE_HOME`` or ``$HOME/.cache``
``xdg-data``          Access the XDG data directory   [#f6]_                    ``$XDG_DATA_HOME`` or ``$HOME/.local/share``
``xdg-run/path``      Access subdirectories of the XDG runtime directory        ``$XDG_RUNTIME_DIR/path`` (``/run/user/$UID/path``)
====================  ========================================================  ===================================================

Except ``host, host-etc, host-os`` paths can be added to all the above
filesystem options. For example, ``--filesystem=xdg-documents/path``.
The following permission options can also be added:

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
.. [#f4] Except ``/app, /dev, /etc, /lib, /lib32, /lib64, /proc, /root, /run/flatpak, /run/host, /sbin, /usr``
.. [#f5] The arbitrary path includes all its subfolders and subfiles if any.
.. [#f6] ``xdg-{cache, config, data}`` binds mount the paths from host to the per-app sandbox directory.
   Inside the sandbox ``$XDG_CACHE_HOME, $XDG_CONFIG_HOME and $XDG_DATA_HOME`` is set to
   ``$HOME/.var/app/<app-id>/{cache, config, data}``. So this permission is not needed
   unless access to the host directory, bind mounted to
   ``$HOME/.var/app/<app-id>/{cache, config, data}`` is desired.

