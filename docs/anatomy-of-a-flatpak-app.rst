Anatomy of a Flatpak Application
================================

Each Flatpak app has the following basic structure:

* ``metadata`` - a keyfile which provides information about the application, including information that is necessary for setting up the sandbox for running the application
* ``/files`` - the files that make up the application
* ``/files/bin`` - application binaries
* ``/export`` - files which the host environment needs access to, such as the application's desktop file, icon and D-Bus service file

A typical metadata file looks like this::

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

This specifies the name of the application, the runtime it requires, the SDK that it is built against and the command used to run it. It also specifies file and device access, sets certain environment variables (inside the sandbox, of course), and how to connect to the session bus.

All the files in the export directory must have the application id as a prefix. This guarantees that applications cannot cause conflicts, and that they can’t override any system installed applications.

AppData
-------

Many Linux distributions provide an app store or app center for browsing and installing applications. `AppData <https://www.freedesktop.org/software/appstream/docs/chap-Quickstart.html#sect-Quickstart-DesktopApps>`_ is a standard format for providing application information that can be used by app stores, such as an application description and screenshots. Flatpak makes use of the AppData standard, and application authors are recommended to use it to include information about their applications.

Extensions
----------

Applications and runtimes can define extension points, where optional pieces can be plugged into the filesystem. Flatpak is using this to separate translations and debuginfo from the main application, and to include certain parts that are provided separately, such as GL libraries or gstreamer plugins.

When flatpak is setting up a sandbox, it is looking for extension points that are declared in the application and runtime metadata, and mounts runtimes with a matching name. A typical extension section in a metadata file looks like this::

  [Extension org.gnome.Platform.GL]
  version=1.4
  directory=lib/GL

More complicated extension points can accept multiple extensions that get mounted below a single directory. For example, the gstreamer extension::

  [Extension org.freedesktop.Platform.GStreamer]
  version=1.4
  directory=lib/extensions/gstreamer-1.0
  subdirectories=true

The ``subdirectories=true`` key instructs flatpak to mount e.g. a ``org.freedesktop.Platform.GStreamer.mp3`` runtime on ``/usr/lib/extensions/gstreamer-1.0/mp3`` in the sandbox. The gstreamer libraries in the ``org.freedesktop.Platform`` runtime have been configured to look in this place for plugins.
