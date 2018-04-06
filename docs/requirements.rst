Requirements & Conventions
==========================

Flatpak deliberately makes as few requirements of applications as possible. However, some adherence to some conventions are necessary to enable integration with Linux desktops and app centers.

Those who have previously targeted the Linux desktop will typically be familiar with these conventions.

Application IDs
---------------

As described in :doc:`using-flatpak`, Flatpak requires each application to have a unique identifier, which has a three part form such as ``org.gnome.Dictionary``. Developers should follow the standard `D-Bus naming conventions <https://dbus.freedesktop.org/doc/dbus-specification.html#message-protocol-names>`_ when creating their own IDs.

Application icons
-----------------

Applications are expected to provide an application icon, which is used for their application launcher. These icons should be provided in accordance with the `Freedesktop icon specification <https://standards.freedesktop.org/icon-theme-spec/icon-theme-spec-latest.html>`_

Icons should be named with the application's ID, be in either PNG or SVG format, and must be placed in the standard location::

  /app/share/icons/hicolor/$size/apps/

For example, the path to the 128✕128px version of GNOME Dictionary's icon is::

  /app/share/icons/hicolor/128x128/apps/org.gnome.Dictionary.png

Desktop files
-------------

Desktop files are another Freedesktop standard which is used to provide information about each application the desktop environment where it is being run. The `Freedesktop specification <https://standards.freedesktop.org/desktop-entry-spec/latest/>`_ provides a complete reference for writing desktop files, and `other useful information about them <https://wiki.archlinux.org/index.php/desktop_entries>`_ are available.

Desktop files should be named with the application's ID followed by the ``.desktop`` file extension, and should be placed in ``/app/share/applications/``. For example::

  /app/share/applications/org.gnome.Dictionary.desktop

A minimal desktop file should contain at least the application's *name*, *exec* command, *type* and *icon* name::

  [Desktop Entry]
  Name=Gnome Dictionary
  Exec=org.gnome.Dictionary
  Type=Application
  Icon=org.gnome.Dictionary

The ``desktop-file-validate`` command can be used to check for errors in desktop files.

AppData files
-------------

AppData files provide metadata about applications, which is used by application stores (such as Flathub, GNOME Software and KDE Discover). It can contain a description of the application, screenshots.

The `Freedesktop specification <https://www.freedesktop.org/software/appstream/docs/>`_ provides a complete reference for providing AppData.

AppData files should be named with the application ID and the ``.appdata.xml`` file extension, and should be placed in ``/app/share/metainfo/``. For example::

  /app/share/metainfo/org.gnome.Dictionary.appdata.xml

The ``appstream-util validate-relax`` command tool can be used to check AppData files for errors.

XDG base directories
--------------------

XDG base directories are `a Freedesktop standard <https://standards.freedesktop.org/basedir-spec/basedir-spec-latest.html>`_, which defines standard locations where user-specific application data and configuration should be stored.

By default, Flatpak sets three XDG base directories that should be used by applications for user-specific storage. These are:

===============  =================================  ======================
Base directory   Usage                              Default location
===============  =================================  ======================
XDG_CONFIG_HOME  User-specific configuration files  ~/.var/<app-id>/config
XDG_DATA_HOME    User-specific data                 ~/.var/<app-id>/data
XDG_CACHE_HOME   Non-essential user-specific data   ~/.var/<app-id>/cache
===============  =================================  ======================

For example, GNOME Dictionary should store user-specific data in::

  ~/.var/org.gnome.Dictionary/data

Note that applications can be configured to use non-default base directory locations (see :doc:`sandbox-permissions`).

Filesystem layout
-----------------

Each application sandbox contains the filesystem of the application's runtime, which follows `standard Linux filesystem conventions <https://en.wikipedia.org/wiki/Filesystem_Hierarchy_Standard>`_. For example:

- ``/bin`` - binaries
- ``/dev`` - device files
- ``/lib`` - libraries
- ``/opt`` - application software packages
- ``/usr`` - multi-user utilities and applications

In addition to this, each sandbox contains a top-level ``/app`` directory, which is where the application's own files are located.

D-Bus
-----

`D-Bus <https://www.freedesktop.org/wiki/Software/dbus/>`_ is the standard framework for interprocess communication on Linux desktops. Applications do not always need to use it, and often portals will be a better choice. However, there are a few cases where D-Bus is required, such as when integrating with system-provided media controls, through the `MPRIS specification <https://specifications.freedesktop.org/mpris-spec/latest/>`_.

If an application provides a D-Bus service, the D-Bus service name is expected to be the same as the application ID.
