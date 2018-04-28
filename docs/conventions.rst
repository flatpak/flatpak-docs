Conventions & Requirements
==========================

Flatpak deliberately makes as few requirements of applications as possible. However, a small number of standard Linux desktop conventions are expected, primarily to ensure that applications integrate with Linux desktops and app centers. These are listed below.

Applications that have previously targeted the Linux desktop will typically need to make very few (if any) changes to conform with these standards.

Application IDs
---------------

As described in :doc:`using-flatpak`, Flatpak requires each application to have a unique identifier, which has a three-part form such as ``org.gnome.Dictionary``. As will be seen below and in future sections, this ID is expected to be used in a number of places. Developers should follow the standard `D-Bus naming conventions <https://dbus.freedesktop.org/doc/dbus-specification.html#message-protocol-names>`_ when creating their own IDs. This format is already recommended by the `Desktop File specification  <https://specifications.freedesktop.org/desktop-entry-spec/desktop-entry-spec-latest.html#file-naming>`_ and `Appstream specification  <https://www.freedesktop.org/software/appstream/docs/chap-Metadata.html#sect-Metadata-GenericComponent>`_ also.

Application icons
-----------------

Applications are expected to provide an application icon, which is used for their application launcher. These icons should be provided in accordance with the `Freedesktop icon specification <https://standards.freedesktop.org/icon-theme-spec/icon-theme-spec-latest.html>`_.

Icons should be named with the application's ID, be in either PNG or SVG format, and must be placed in the standard location::

  /app/share/icons/hicolor/$size/apps/

For example, the path to the 128✕128px version of GNOME Dictionary's icon is::

  /app/share/icons/hicolor/128x128/apps/org.gnome.Dictionary.png

Desktop files
-------------

Desktop files are another Freedesktop standard, which is used to provide the desktop environment with information about each application. The `Freedesktop specification <https://standards.freedesktop.org/desktop-entry-spec/latest/>`_ provides a complete reference for writing desktop files, and `additional information about them <https://wiki.archlinux.org/index.php/desktop_entries>`_ is available online.

Desktop files should be named with the application's ID, followed by the ``.desktop`` file extension, and should be placed in ``/app/share/applications/``. For example::

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

AppData files provide metadata about applications, which is used by application stores (such as Flathub, GNOME Software and KDE Discover). The `Freedesktop AppStream specification <https://www.freedesktop.org/software/appstream/docs/>`_ provides a complete reference for providing AppData.

AppData files should be named with the application ID and the ``.appdata.xml`` file extension, and should be placed in ``/app/share/metainfo/``. For example::

  /app/share/metainfo/org.gnome.Dictionary.appdata.xml

The ``appstream-util validate-relax`` command can be used to check AppData files for errors.

XDG base directories
--------------------

`XDG base directories <https://standards.freedesktop.org/basedir-spec/basedir-spec-latest.html>`_ are a Freedesktop standard which defines standard locations where user-specific application data and configuration should be stored. If your application already respects these nothing must be changed.

By default, Flatpak sets three XDG base directories that should be used by applications for user-specific storage. These are:

===============  =================================  ==========================
Base directory   Usage                              Default location
===============  =================================  ==========================
XDG_CONFIG_HOME  User-specific configuration files  ~/.var/app/<app-id>/config
XDG_DATA_HOME    User-specific data                 ~/.var/app/<app-id>/data
XDG_CACHE_HOME   Non-essential user-specific data   ~/.var/app/<app-id>/cache
===============  =================================  ==========================

For example, GNOME Dictionary will store user-specific data in::

  ~/.var/org.gnome.Dictionary/data/gnome-dictionary

Note that applications can be configured to use non-default base directory locations (see :doc:`sandbox-permissions`).

Filesystem layout
-----------------

Each Flatpak sandbox, which is the environment in which an application is run, contains the filesystem of the application's runtime. This follows `standard Linux filesystem conventions <https://en.wikipedia.org/wiki/Filesystem_Hierarchy_Standard>`_.

For example, the root of the sandbox contains the ``/etc`` directory for configuration files and ``/usr`` for multi-user utilities and applications. In addition to this, each sandbox contains a top-level ``/app`` directory, which is where the application's own files are located.

D-Bus
-----

Flatpak supports `D-Bus <https://www.freedesktop.org/wiki/Software/dbus/>`_, the standard framework for inter-process communication on Linux desktops. This can be used for application launching and communicating with some system services. Applications can also provide their own D-Bus services (when doing this, the D-Bus service name is expected to be the same as the application ID).
