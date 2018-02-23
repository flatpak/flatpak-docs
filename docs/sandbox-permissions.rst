Sandbox Permissions
===================

Sandbox permissions can be configured from an application manifest (see :doc:`manifests`). They can also be set with the ``build-finish``, ``run`` and ``override`` commands.

Permission options
------------------

The following list includes some of the most useful permission options. The full list can be viewed using ``flatpak build-finish --help``

===================================================  ===========================================
``--filesystem=host``                                Access all files
``--filesystem=home``                                Access the home directory
``--filesystem=home:ro``                             Access the home directory, read-only
``--filesystem=/some/dir --filesystem=~/other/dir``  Access paths
``--filesystem=xdg-download``                        Access the XDG download directory
``--nofilesystem=...``                               Undo some of the above
``--socket=x11 --share=ipc``                         Show windows using X11 [#f1]_
``--device=dri``                                     OpenGL rendering
``--socket=wayland``                                 Show windows using Wayland
``--socket=pulseaudio``                              Play sounds using PulseAudio
``--share=network``                                  Access the network [#f2]_
``--talk-name=org.freedesktop.secrets``              Talk to a named service on the session bus
``--system-talk-name=org.freedesktop.GeoClue2``      Talk to a named service on the system bus
``--socket=system-bus``                              Unlimited access to all of D-Bus
===================================================  ===========================================

.. note::
  Until a sandbox-compatible backend is available, access to dconf needs to be enabled using the following options::

    --filesystem=xdg-run/dconf
    --filesystem=~/.config/dconf:ro
    --talk-name=ca.desrt.dconf
    --env=DCONF_USER_CONFIG_DIR=.config/dconf

.. rubric:: Footnotes

.. [#f1] ``--share=ipc`` means that the sandbox shares IPC namespace with the host. This is not necessarily required, but without it the X shared memory extension will not work, which is very bad for X performance.
.. [#f2] Giving network access also grants access to all host services listening on abstract Unix sockets (due to how network namespaces work), and these have no permission checks. This unfortunately affects e.g. the X server and the session bus which listens to abstract sockets by default. A secure distribution should disable these and just use regular sockets.

