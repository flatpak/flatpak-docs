Elements of a Flatpak Application
=================================

Flatpak expects applications to follow standard Linux desktop conventions. These are supplemented with a small number of Flatpak-specific elements that are used to distribute, install and run applications.

Standard application elements
-----------------------------

The following are some of the Linux desktop conventions that are supported and expected by Flatpak. Application developers are encouraged to use them.

* `AppData <https://www.freedesktop.org/software/appstream/docs/chap-Quickstart.html#sect-Quickstart-DesktopApps>`_, for providing information that can be used by app stores, such as an application description and screenshots
* Application icons, provided in the typical sizes, formats and locations
* D-Bus service files, for D-Bus activation and any services provided by the application
* .desktop files, for providing basic information about the application

Application structure
---------------------

When an application is built using flatpak, it is outputted with the following structure:

* ``metadata`` - a keyfile which provides information about the application
* ``/files`` - the files that make up the application, include source code and application data
* ``/files/bin`` - application binaries
* ``/export`` - files which the host environment needs access to, such as the application's AppData, .desktop file, icon and D-Bus service file

All the files in the export directory must have the application ID as a prefix. This guarantees that applications cannot cause conflicts, and that they can’t override any system installed applications.

Metadata files
--------------

The application's ``metadata`` file provides information that allows flatpak to set up the sandbox for running the application. A typical metadata file looks like this::

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

This specifies the name of the application, the runtime it requires, the SDK that it is built against and the command used to run it. It also specifies file and device access, sets certain environment variables (inside the sandbox, of course), and how to connect to the session bus. Details on how to change these metadata parameters are included in subsequent sections.

.. note::
  While it is most common to encounter application metadata files, runtimes and extensions also have them.
