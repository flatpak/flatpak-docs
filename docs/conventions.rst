Requirements & Conventions
==========================

Flatpak deliberately imposes very few requirements on applications.
However, a small number of standard Linux desktop conventions
are expected, primarily to ensure that applications integrate with Linux
desktops and app centers. Developers might also encounter a small number of
Linux technical conventions.

Information on additional desktop integration options can be found in
:doc:`desktop-integration`.

Expected Standards
------------------

Applications using Flatpak are generally expected to comply with the
following standards. Applications that have previously targeted the Linux
desktop will typically need to make very few (if any) changes to meet these requirements.

Application IDs
```````````````

As described in :doc:`using-flatpak`, Flatpak requires each application to have a
unique identifier, which follows a format such as ``org.example.app``.

The format is in reverse-DNS style, meaning the first section should be a domain
controlled by the project, while the trailing section represents the specific project.
There are exceptions to this, such as extensions, which use the base application-id of the project
they extend rather than their own.

As will be seen below and in future sections, this ID is expected to be used in a number of places.
Developers must follow the standard `D-Bus naming conventions for bus names
<https://dbus.freedesktop.org/doc/dbus-specification.html#message-protocol-names>`_
when creating their own IDs. This format is
already recommended by the `Desktop File specification
<https://specifications.freedesktop.org/desktop-entry-spec/latest/file-naming.html>`_
and also the `Appstream specification
<https://www.freedesktop.org/software/appstream/docs/chap-Metadata.html#sect-Metadata-GenericComponent>`_.

Here are some practical examples of poor ID choices:

- ``org.example.desktop``

  This ID is problematic because the Appstream standard, for legacy reasons, treats IDs ending with
  ``.desktop`` as a special case, leading to inconsistency. For the same reason, ``.Desktop`` suffixes
  should not be used for newly named applications. Don't hesitate to repeat the application name
  even if it is already part of the domain name section of the identifier (e.g. ``org.example.Example``).

- ``io.github.foo``

  This is problematic because while ``foo.github.io`` may be unique to your project, it does not
  include a project-specific identifier. For instance, if another project
  creates ``io.github.foo-bar``, issues can arise since the two IDs would then
  be treated as similar by ``flatpak`` (even though they deserve to be treated as different
  namespaces). A better ID would be ``io.github.foo.foo``, even if the second
  ``foo`` seems redundant.

- ``org.example-site.foo``

  This ID is not valid according to the D-Bus specification, as the dash
  ``-`` isn't allowed except in the last component. You should replace
  the dash with an underscore ``_``, and thus use
  ``org.example_site.Foo`` instead.

- ``com.github.foo.bar`` or ``com.gitlab.foo.bar``

  While a project may be hosted on GitHub or GitLab, it does not have
  control over the ``github.com`` or ``gitlab.com``
  domains. Instead, you should use ``io.github`` or ``io.gitlab`` as
  shown above.

- ``Org.Example.App``

  The domain portion of the ID must be in lowercase, and while not required,
  the application portion is recommended to be in lowercase as well.
  Therefore, you should use ``org.example.app``.

MetaInfo files
``````````````

MetaInfo files provide metadata about applications, which is
used by application stores (such as Flathub, GNOME Software
and KDE Discover).

The `Freedesktop AppStream specification
<https://www.freedesktop.org/software/appstream/docs/>`_ provides a complete
reference for providing MetaInfo. You can use the online
`AppStream MetaInfo Creator <https://www.freedesktop.org/software/appstream/metainfocreator/>`_
to generate a basic file.

MetaInfo files should be named using the application ID, must end with the ``.metainfo.xml``
file extension, and must be placed in ``/app/share/metainfo/``. For example::

  /app/share/metainfo/org.gnome.Dictionary.metainfo.xml

A legacy convention allows for placing the ``.appdata.xml`` file in ``/app/share/appdata``.
``flatpak-builder`` will check either directory for
its respective extension.

The ``appstreamcli validate --explain`` command can be used to check MetaInfo
files for errors.

The metainfo file along with the other metadata such as icon and desktop
files is composed into a catalogue by ``appstream``. Since 1.3.4,
Flatpak Builder by default uses ``appstreamcli`` from `libappstream <https://github.com/ximion/appstream/>`_
to compose this and executes the following command::

  $ appstreamcli compose --no-partial-urls --prefix=/ --origin=${FLATPAK_ID} --media-baseurl=<repo-media-url> --media-dir=${FLATPAK_DEST}/share/app-info/media --result-root=${FLATPAK_DEST} --data-dir=${FLATPAK_DEST}/share/app-info/xmls --icons-dir=${FLATPAK_DEST}/share/app-info/icons/flatpak --components=${FLATPAK_ID} ${FLATPAK_DEST}

``${FLATPAK_ID}`` is the Flatpak application ID and ``${FLATPAK_DEST}``
is set to ``/app`` for apps and ``/usr`` for runtimes. The ``media-baseurl``
controls the base URL used for mirroring screenshots. Usually each app
store will have their own base URL for this. On Flathub this is
set to ``https://dl.flathub.org/media/``.

Application icons
`````````````````

Applications are expected to provide an application icon, which
is used for their application launcher. These icons should be
provided in accordance with the `Freedesktop icon specification
<https://specifications.freedesktop.org/icon-theme-spec/icon-theme-spec-latest.html>`_.

Icons must be named using the application's ID, be in either PNG or SVG
format, and must be placed in the standard location::

  /app/share/icons/hicolor/$size/apps/

For example, the path to the 128✕128px version of GNOME Dictionary's
icon is::

  /app/share/icons/hicolor/128x128/apps/org.gnome.Dictionary.png

Icons must be square-shaped, meaning their width and height must be the
same. The maximum size allowed by the specification is 512x512px. SVG
icons are of size ``scalable``::

  /app/share/icons/hicolor/scalable/apps/org.gnome.Dictionary.svg

Flatpak will export the following icon name patterns:
``$FLATPAK_ID``, ``$FLATPAK_ID.foo``, ``$FLATPAK_ID-foo``. They may end with an
extension suffix like ``.png`` or ``.svg``. Exported icons can be found in the
``icons`` subfolder of either ``$HOME/.local/share/flatpak/exports/share`` or
``/var/lib/flatpak/exports/share``, depending on whether it's a system or user install.

The distribution usually appends those paths to ``$XDG_DATA_DIRS`` on
the host when installing the ``flatpak`` package. Unless an icon is exported
by Flatpak, host applications cannot access it.

Desktop files
`````````````

Desktop files are used to provide the desktop environment with
information about each application. The `Freedesktop specification
<https://specifications.freedesktop.org/desktop-entry-spec/latest/>`_
provides a complete reference for writing desktop files.

Desktop files must be named using the application's ID, followed
by the ``.desktop`` file extension, and must be placed in
``/app/share/applications/``. For example::

  /app/share/applications/org.gnome.Dictionary.desktop

A minimal desktop file should contain at least the application's *name*,
*exec* command, *type*, *icon* name and *categories*::

  [Desktop Entry]
  Name=Gnome Dictionary
  Exec=gnome-dictionary
  Type=Application
  Icon=org.gnome.Dictionary
  Categories=Office;Dictionary;

The ``desktop-file-validate`` command can be used to check for errors in
desktop files.

The ``Exec`` key of the desktop files is rewritten by Flatpak when installing
an app. The original value of the key becomes the value of the ``--command``
argument like so::

  Exec=/usr/bin/flatpak run --branch=stable --arch=x86_64 --command=gnome-dictionary org.gnome.Dictionary

Flatpak will export the following desktop filename patterns:
``$FLATPAK_ID.desktop``, ``$FLATPAK_ID.foo.desktop``, ``$FLATPAK_ID-foo.desktop``.
Exported desktop files can be found in the ``applications`` subfolder of either
``$HOME/.local/share/flatpak/exports/share`` or
``/var/lib/flatpak/exports/share``, depending on whether it's a system or
user install.

The distribution usually appends those paths to ``$XDG_DATA_DIRS`` on
the host when installing the ``flatpak`` package. Unless a desktop file is
exported by Flatpak, host applications cannot access it.

D-Bus service files
````````````````````

D-Bus service files are sometimes supplied by applications for
automatic "on demand" activation of services such as when setting up an
appliction for D-Bus launching.

The service file must be installed to ``${FLATPAK_DEST}/share/dbus-1/services``
and must end in ``.service``. The rest of the filename must match
the ``$FLATPAK_ID`` or a subname of the Flatpak ID
(``$FLATPAK_ID.foo``).

The value of the ``Name`` key inside the service file must match
the filename without the ``.service`` part exactly. An example is
provided below::

  # Installed as /app/share/dbus-1/services/org.example.coolapp.service

  [D-BUS Service]
  Name=org.example.coolapp
  Exec=/app/bin/coolapp --gapplication-service

  # Installed as /app/share/dbus-1/services/org.example.coolapp.foobar.service

  [D-BUS Service]
  Name=org.example.coolapp.foobar
  Exec=/app/bin/coolapp --arg1 --gapplication-service

GNOME shell search providers
`````````````````````````````

A GNOME Shell search provider is a mechanism by which an application
can expose its search capabilities to the GNOME Shell. Note that
Flatpak will mark all search providers files as disabled when exporting
them.

The search provider file must be installed as
``${FLATPAK_DEST}/share/gnome-shell/search-providers/$NAME-search-provider.ini``

``$NAME`` can be either the ``$FLATPAK_ID`` or a subname of the Flatpak
ID (``$FLATPAK_ID.foo``, ``$FLATPAK_ID-foo``).

An example is provided below::

  # Installed as /app/share/gnome-shell/search-providers/org.example.coolapp-search-provider.ini

  [Shell Search Provider]
  DesktopId=org.example.coolapp.desktop
  BusName=org.example.coolapp.SearchProvider
  ObjectPath=/org/mozilla/coolapp/SearchProvider
  Version=2

Krunner DBus plugins
`````````````````````

Krunner DBus plugins offer similar functionality in KDE as GNOME shell
search providers. Flatpak supports exporting these since 1.16.0 and
they are also disabled by default.

The plugin file must be installed as
``${FLATPAK_DEST}/share/krunner/dbusplugins/$NAME.desktop``. ``$NAME``
can be either the ``$FLATPAK_ID`` or a subname of the Flatpak ID
(``$FLATPAK_ID.foo``, ``$FLATPAK_ID-foo``).

An example is provided below::

  # Installed as /app/share/krunner/dbusplugins/org.example.coolapp.desktop

  [Desktop Entry]
  Name=Hello
  X-KDE-ServiceTypes=Plasma/Runner
  Type=Service
  Icon=org.example.coolapp
  X-KDE-ServiceTypes=Plasma/Runner
  X-KDE-PluginInfo-EnabledByDefault=true
  X-Plasma-API=DBus
  X-Plasma-DBusRunner-Service=org.example.coolapp.KRunner
  X-Plasma-DBusRunner-Path=/org/example/coolapp/KRunner

MIME files
``````````

MIME files define new MIME types, file extensions, and detection rules.
The MIME file must be installed as
``${FLATPAK_DEST}/share/mime/packages/$NAME.xml``. ``$NAME``
can be either the ``$FLATPAK_ID`` or a subname of the Flatpak ID
(``$FLATPAK_ID.foo``, ``$FLATPAK_ID-foo``). It is common to use
``$FLATPAK_ID-mime.xml`` as the filename.

Note, that Flatpak may rewrite these MIME files to remove magic mime
rules and drop globs to a lower priority. An example MIME file of
`Akonadi <https://github.com/KDE/akonadi/blob/master/akonadi-mime.xml>`_
is provided below::

  <?xml version="1.0" encoding="UTF-8"?>
  <!--
  SPDX-License-Identifier: GPL-2.0-or-later
  -->
  <!--
  Notes:
  - the mime types in this file are valid with the version 0.20 of the
    shared-mime-info package.
  - the "fdo #xxxxx" are the wish in the freedesktop.org bug database to include
    the mime type there.
  -->
  <mime-info xmlns="http://www.freedesktop.org/standards/shared-mime-info">
    <mime-type type="application/x-vnd.akonadi.calendar.event">
      <sub-class-of type="text/calendar"/>
      <comment>iCal Calendar Event Component</comment>
    </mime-type>
    <mime-type type="application/x-vnd.akonadi.calendar.freebusy">
      <sub-class-of type="text/calendar"/>
      <comment>iCal Calendar FreeBusy Component</comment>
    </mime-type>
    <mime-type type="application/x-vnd.akonadi.calendar.journal">
      <sub-class-of type="text/calendar"/>
      <comment>iCal Calendar Journal Component</comment>
    </mime-type>
    <mime-type type="application/x-vnd.akonadi.calendar.todo">
      <sub-class-of type="text/calendar"/>
      <comment>iCal Calendar TODO Component</comment>
    </mime-type>
    <mime-type type="application/x-vnd.akonadi.collection.virtual">
      <comment>Virtual Akonadi Collection</comment>
    </mime-type>
  </mime-info>

This is installed as ``/app/share/mime/packages/org.kde.akregator.xml``
in Akregator.

Technical conventions
---------------------

The following are standard technical conventions used by Flatpak and
Linux desktops. Those with Linux experience will likely already be aware
of them. However, developers who are new to Linux might find some of this
information useful.

D-Bus
`````

D-Bus is the standard IPC framework used on Linux desktops. A lot of
applications won't need to use it, but it is supported by Flatpak should it
be required.

D-Bus can be used for application launching and communicating with some system
services. Applications can also provide their own D-Bus services (when doing
this, the D-Bus service name is expected to be the same as the application ID).

Filesystem layout
`````````````````

Each Flatpak sandbox, which is the environment in which an
application is run, contains the filesystem of the application's
runtime. This follows `standard Linux filesystem conventions
<https://en.wikipedia.org/wiki/Filesystem_Hierarchy_Standard>`_.

For example, the root of the sandbox contains the ``/etc`` directory for
configuration files and ``/usr`` for multi-user utilities and applications. In
addition to these, each sandbox contains a top-level ``/app`` directory,
which is where the application's own files are located.

XDG base directories
--------------------

`XDG base directories
<https://specifications.freedesktop.org/basedir-spec/basedir-spec-latest.html>`_ are
standard locations for user-specific application data. Popular toolkits provide
convenience functions for accessing XDG base directories. These include:

- Electron: XDG base directories can be accessed with ``app.getPath``.
- Glib: XDG base directories can be accessed through
  the ``g_get_user_cache_dir ()``, ``g_get_user_data_dir ()``,
  ``g_get_user_config_dir ()`` functions.
- Qt: XDG base directories can be accessed with the `QStandardPaths
  Class <https://doc.qt.io/archives/qt-5.15/qstandardpaths.html>`_.

However, applications that aren't using one of these toolkits can expect to
find their XDG base directories in the following locations:

===============  =================================  ================================
Base directory   Usage                              Default location
===============  =================================  ================================
XDG_CONFIG_HOME  User-specific configuration files  ~/.var/app/<app-id>/config
XDG_DATA_HOME    User-specific data                 ~/.var/app/<app-id>/data
XDG_CACHE_HOME   Non-essential user-specific data   ~/.var/app/<app-id>/cache
XDG_STATE_HOME   State data such as undo history    ~/.var/app/<app-id>/.local/state
===============  =================================  ================================

For example, GNOME Dictionary will store user-specific data in::

  ~/.var/app/org.gnome.Dictionary/data/gnome-dictionary

These environment variables are always set by Flatpak and override any host values.
However, if access to directories on the host is needed, the ``$HOST_XDG_CONFIG_HOME``,
``$HOST_XDG_DATA_HOME``, ``$HOST_XDG_CACHE_HOME``, and ``$HOST_XDG_STATE_HOME`` environment
variables will be set if they are also available on the host.

Note that ``$XDG_STATE_HOME`` and ``$HOST_XDG_STATE_HOME`` are only supported by Flatpak 1.13
and later. If your application needs to work on earlier versions of Flatpak, you can use the
``--persist=.local/state`` and ``--unset-env=XDG_STATE_HOME`` finish args, ensuring
the app will use the correct directory, even after Flatpak is later upgraded to version
>1.13.

Also, note that applications can be configured to use non-default base directory
locations (see :doc:`sandbox-permissions`).
