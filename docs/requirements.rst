Requirements & Conventions
==========================

Flatpak deliberately makes as few requirements of applications as possible. However, some adherence to some conventions are necessary to enable integration with Linux desktops and app centers.

Applications that have previously targetted the Linux desktop will typically be familiar with these conventions.

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
