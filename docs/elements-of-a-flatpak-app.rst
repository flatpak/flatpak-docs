Elements of a Flatpak Application
=================================

Flatpak expects applications to follow standard Linux desktop conventions. These are supplemented with a small number of Flatpak-specific elements that are used to distribute, install and run applications.

Standard application elements
-----------------------------

The following are some of the Linux desktop conventions that are supported and expected by Flatpak. Application developers are encouraged to use them.

* `AppData <https://www.freedesktop.org/software/appstream/docs/chap-Quickstart.html#sect-Quickstart-DesktopApps>`_, for providing application information, such as descriptions and screenshots, that is used by app stores
* Application icons, as specified by the `Freedesktop icon theme specification <https://standards.freedesktop.org/icon-theme-spec/icon-theme-spec-latest.html>`_
* `D-Bus <https://www.freedesktop.org/wiki/Software/dbus/>`_, for interaction with the host
* `Desktop files <https://standards.freedesktop.org/desktop-entry-spec/latest/>`_, for providing basic information about the application
* `PulseAudio <https://www.freedesktop.org/wiki/Software/PulseAudio/>`_, for sound
* `X11 <https://www.x.org/wiki/>`_ and `Wayland <https://wayland.freedesktop.org/>`_, for display

Application structure
---------------------

When an application is built using flatpak, it is outputted with the following structure:

* ``metadata`` - a keyfile which provides information about the application
* ``/files`` - the files that make up the application, include source code and application data
* ``/files/bin`` - application binaries
* ``/export`` - files which the host environment needs access to, such as the application's AppData, .desktop file, icon and D-Bus service files

All the files in the export directory must have the application ID as their prefix. For example: 

* ``org.gnome.App.appdata.xml``
* ``org.gnome.App.desktop``
* ``org.gnome.App.png``
* ``org.gnome.App.service``

Naming files in this way prevents naming conflicts and ensures that system installed applications aren't overwritten.

To name exported files in this way, either rename the relevant source files or use flatpak-builder to rename the files at build time (this is explained in more detail in :doc:`flatpak-builder`).

Metadata files
--------------

An application's ``metadata`` file provides information that allows flatpak to set up the sandbox for running the application. A typical metadata file looks like this::

  [Application]
  name=org.gnome.gedit
  runtime=org.gnome.Platform/x86_64/3.22
  sdk=org.gnome.Sdk/x86_64/3.22
  command=gedit

  [Context]
  shared=ipc;network;
  sockets=x11;wayland;pulseaudio;
  devices=dri;
  filesystems=host;

  [Environment]
  GEDIT_FOO=bar

  [Session Bus Policy]
  org.extra.name=talk
  org.other.name=own

This specifies the name of the application, the runtime it requires, the SDK that it was built against and the command used to run it. It also specifies file and device access, sets certain environment variables (inside the application's sandbox, of course), and how it connects to the session bus. Details on how to change these metadata parameters are included in subsequent sections.

.. note::
  While it is most common to encounter metadata files for applications, runtimes and extensions also have them.
