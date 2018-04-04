Freedesktop quick reference
===========================

In order to ensure interoperability, flatpak adheres strictly to a
number of freedesktop standards and practices. This page describes the
basic conventions that should be followed when building a flatpak app.

Icons
-----

Application icons can be in either png or svg format, must use the
application's appid as a prefix and be placed in
``/app/share/icons/hicolor/$size/apps/``

Example:

::

    /app/share/icons/hicolor/128x128/apps/org.gnome.Dictionary.png

If interested, you can read the full spec
`here <https://standards.freedesktop.org/icon-theme-spec/icon-theme-spec-latest.html>`__

Desktop files
-------------

Desktop files are used by desktop environments in order to identify and
display available applications to the user, they contain information
about how to launch the application, its icon and categories among
others.

A minimal desktop file needs at least the application's *Name*, *Exec*
command, *Type* and *Icon*:

::

    [Desktop Entry]
    Name=Gnome Dictionary
    Exec=org.gnome.Dictionary
    Type=Application
    Icon=org.gnome.Dictionary

Your desktop file should be prefixed with your application's appid and
placed in ``/app/share/applications/``, you should use
``desktop-file-validate`` to check your file for errors before including
it.

Example:

::

    /app/share/applications/org.gnome.Dictionary.desktop

You can find more information
`here <https://wiki.archlinux.org/index.php/desktop_entries>`__. If
interested, you can read also the full spec
`here <https://standards.freedesktop.org/desktop-entry-spec/latest/>`__.

Appdata files
-------------

Appdata files are used by application stores (eg. kde discover, gnome
software) in order to display metadata about your application, such as a
description, screenshots, display changelogs on update, among other
things.

your desktop file should be prefixed with your application's appid and
placed in ``/app/share/metainfo/``, you should also use
``appstream-util validate-relax`` to check your file for errors before
including it.

Example:

::

    /app/share/metainfo/org.gnome.Dictionary.appdata.xml

If interested, you can read the full spec
`here <https://www.freedesktop.org/software/appstream/docs/>`__
