Working with the Sandbox
========================

By default, a flatpak has extremely limited access to the host environment. This includes:

* No access to any host files except the runtime, the app and ``~/.var/app/$APPID``. Only the last of these is writable.
* No access to the network.
* No access to any device nodes (apart from ``/dev/null``, etc).
* No access to processes outside the sandbox.
* Limited syscalls.  For instance, apps can't use nonstandard network socket types or ptrace other processes.
* Limited access to the session D-Bus instance - an app can only own its own name on the bus.
* No access to host services like X, system D-Bus, or PulseAudio.

Most applications will need access to some of these resources in order to be useful, and flatpak provides a number of ways to give an application access to them. The ``build-finish`` command is the simplest of these. As was seen in a previous example, this can be used to add access to graphics sockets and network resources::

  $ flatpak build-finish dictionary2 --socket=x11 --share=network --command=gnome-dictionary

These arguments translate into several properties in the application metadata file::

  [Application]
  name=org.gnome.Dictionary
  runtime=org.gnome.Platform/x86_64/3.22
  sdk=org.gnome.Sdk/x86_64/3.22
  command=gnome-dictionary

  [Context]
  shared=network;
  sockets=x11;

Note that in this example access to the filesystem wasn't granted. This can be tested by installing the resulting application and running::

  $ flatpak run --command=ls org.gnome.Dictionary ~/
  
build-finish allows a whole range of resources to be added to an application. Run ``flatpak build-finish --help`` to view the full list.

There are several ways to override the permissions that are set in an application's metadata file. One of these is to override them using flatpak run, which accepts the same parameters as ``build-finish``. For example, this will let the Dictionary application see your home directory::

  $ flatpak run --filesystem=home --command=ls org.gnome.Dictionary ~/
  
``flatpak run`` can also be used to permanently override an application's permissions::

  $ flatpak --user override --filesystem=home org.gnome.Dictionary
  $ flatpak run --command=ls org.gnome.Dictionary ~/
  
It is also possible to remove permissions using the same method. You can use the following command to see what happens when access to the filesystem is removed, for example::

  $ flatpak run --nofilesystem=home --command=ls org.gnome.Dictionary ~/

Useful sandbox permissions
--------------------------

Flatpak provides an array of options for controlling sandbox permissions. The following are some of the most useful.

===============================================  ===========================================
--filesystem=host                                Access all files
--filesystem=home                                Access the home directory
--filesystem=home:ro                             Access the home directory, read-only
--filesystem=/some/dir --filesystem=~/other/dir  Access paths
--filesystem=xdg-download                        Access the XDG download directory
--nofilesystem=...                               Undo some of the above
--socket=x11 --share=ipc                         Show windows using X11 [#f1]_
--device=dri                                     OpenGL rendering
--socket=wayland                                 Show windows using Wayland
--socket=pulseaudio                              Play sounds using PulseAudio
--share=network                                  Access the network [#f2]_
--talk-name=org.freedesktop.secrets              Talk to a named service on the session bus
--system-talk-name=org.freedesktop.GeoClue2      Talk to a named service on the system bus
--socket=system-bus --socket=session-bus         Unlimited access to all of D-Bus
===============================================  ===========================================

.. rubric:: Footnotes

.. [#f1] ``–share=ipc`` means that the sandbox shares IPC namespace with the host. This is not necessarily required, but without it the X shared memory extension will not work, which is very bad for X performance.
.. [#f2] Giving network access also grants access to all host services listening on abstract Unix sockets (due to how network namespaces work), and these have no permission checks. This unfortunately affects e.g. the X server and the session bus which listens to abstract sockets by default. A secure distribution should disable these and just use regular sockets.

