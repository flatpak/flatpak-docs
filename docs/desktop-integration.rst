Desktop Integration
===================

:doc:`conventions` covers the essential aspects of Linux desktop
integration. This page provides further information on optional desktop
integration features. It also provides guidance on how applications can
ensure that their user interfaces fit into the whole range of Linux desktops
and distributions.

This information is primarily intended for developers who are new to
Linux. However it is also relevant to desktop-specific Linux applications
who wish to target a broader range of Linux environments.

While targeting the Linux desktop ecosystem might seem challenging, the
existence of common standards, in combination with these guidelines, means
that supporting the full range of Linux environments needn't be difficult.

Locale detection
----------------

Application toolkits, such as Electron, GTK and Qt, provide built-in support
for detecting which locale to use. Otherwise, the ``setlocale`` function
can be used.

Portals
-------

Portals are the framework for securely accessing resources from outside an
application sandbox. They provide a range of common features to applications,
including:

- Determining network status
- Opening a file with a file chooser
- Opening URIs
- Preventing the device from suspend/sleep/powering off
- Printing
- Sending email
- Showing notifications
- Taking screenshots and screencasts

Toolkits like GTK and Qt provide transparent support for portals.

.. toctree::
  :maxdepth: 1

  portals

If you are not using one of these toolkits, it is possible to access
the portals API directly. See the `Portals API documentation
<https://flatpak.github.io/xdg-desktop-portal/>`_ for more
information.

Notifications
-------------

A number of toolkits and frameworks provide transparent support for Linux
desktop notifications. This includes Electron, GTK, KDE and QML.

Status icons
------------

Status icons are the same concept as the system tray or the taskbar on Windows,
or menu bar icons on Mac. These are supported on most Linux distributions,
through abstractions such as libappindicator.

A number of Linux distributions don't show status icons. It is still possible
to provide a status icon, and it will be shown in some distributions. However,
in order to ensure compatibility, it is recommended to only use status icons
in a supplementary manner, and not to rely on them as the only mechanism for
providing status information or access to particular features. This includes
"minimize to tray" (or equivalent) functionality.

XEmbed style icons will function on desktops that support them with the
``x11`` permission.

StatusNotifier
^^^^^^^^^^^^^^

StatusNotifier style icons will not function without extra permissions as it
requires talking to a non-hardenend host service. Risks include impersonation of
other software and exploitation of bugs in the host service such as image decoders.

At the very minimum to use StatusNotifier you must have the
``--talk-name=org.kde.StatusNotifierWatcher`` permission to register an item.

Depending on the exact implementation of StatusNotifier that your application is
using it may need session bus ownership of ``org.kde.StatusNotifierItem-$PID-$ITEM_ID``.

This permission is problematic in Flatpak as the ``$PID`` value is often the same
in sandboxes and the item will possibly conflict with other applications.
However if needed the ``--own-name=org.kde.*`` permission will allow this. This opens
many new risks including the ability to impersonate any KDE service or application
possibly capturing important user data.

Most implementations of StatusNotifer have dropped this requirement but known exceptions
to this are Electron versions older than 23.3.0.

Current versions of Electron, Chromium, KNotifications, and libappindicator are known
to work without ownership permissions.

System search
-------------

GNOME-based distributions, like CentOS, Fedora, Red Hat Enterprise Linux and
Ubuntu, provide the option to integrate with system search, by providing a
`search provider <https://developer.gnome.org/documentation/tutorials/search-provider.html>`_. This allows
application-provided search results to appear in the Activities Overview.

Window controls
---------------

Window controls are the buttons used to close, maximize and minimize
windows. These do vary across Linux desktops, particularly in terms of which
controls are shown. Whether applications attempt to follow these variations
is up to their discretion. Providing the exact same controls as used by a
particular desktop environment should not be seen as a hard requirement.

From a user experience perspective, it is important to ensure that window
controls appear on the same side of the window as other desktops. On Linux
this is the right side of the window (like Windows).

On X11 applications can rely on system-provided titlebars if they don't
want to draw their own window controls. For a consistent Wayland experience
applications must always provide their own. Typically toolkits handle this
but raw wayland clients can use `libdecor <https://gitlab.gnome.org/jadahl/libdecor>`_ for a
general solution.

Window decorations
------------------

If your application uses a dark visual style as well as system-provided window
decorations, the ``GTK_THEME_VARIANT=dark`` X11 window property should be
used, to ensure that window decorations match the rest of the application
window. This can be done by running::

  xprop -f _GTK_THEME_VARIANT 8u -set _GTK_THEME_VARIANT dark

Global menus
------------

If your application uses `Gtk.Application:menubar <https://docs.gtk.org/gtk4/class.Application.html#properties>`_ or `QMenuBar <https://doc.qt.io/qt-6/qmenubar.html>`_
it will work as expected from within a sandboxed application.

Theming
-------

Flatpak applications cannot directly use the system theme. This happens because flatpaks do not have the ability to use data files or libraries in ``/usr`` (where system themes are typically located). The solution to this is to package themes as Flatpaks, as relying upon the host to have the correct version for every flatpak defeats the portability benefits it provides. These themes are provided as :doc:`extension`, to the Freedesktop runtime for Gtk3 themes, and to the KDE runtime for Qt themes.

The theming system requires Flatpak 0.8.4+ and applications using up to date ``org.gnome.Platform`` 3.24+, or ``org.freedesktop.Platform`` 1.6+, or ``org.kde.Platform`` 5.9+.

Installing themes
^^^^^^^^^^^^^^^^^

Instructions for Gtk themes
```````````````````````````

As of Flatpak 0.10.1, Flatpak can automatically detect the active Gtk
theme on host by reading the value of the ``gtk-theme`` DConf key.

If the corressponding theme is packaged as an extension in the remote,
Flatpak will automatically install it during ``flatpak install`` or
``flatpak update``.

Instructions for Qt themes
``````````````````````````

There are a few Qt theme extensions packaged on Flathub. To see a list,
you can run::

 $ flatpak remote-ls --columns=application flathub | grep org.kde.KStyle

Then you can install the theme with::

 $ flatpak install flathub org.kde.KStyle.Foo

replacing ``Foo`` with the name of the desired theme. The theme needs
to be available for the KDE runtime branch used by the application.

Gtk look and feel in Qt applications
````````````````````````````````````

Since ``org.kde.Platform`` runtime branches 5.15-22.08+ and 6.5+, on
Wayland, ``org.kde.WaylandDecoration.QAdwaitaDecorations`` and
``org.kde.WaylandDecoration.QGnomePlatform-decoration`` needs to be
installed. Please see the `usage <https://github.com/FedoraQt/QAdwaitaDecorations?tab=readme-ov-file#usage>`_
instructions upstream too.

These plugins will be part of upstream starting at Qt 6.8.

For older runtimes, ``org.kde.PlatformTheme.QGnomePlatform`` and
``org.kde.WaylandDecoration.QGnomePlatform-decoration`` needs to be
installed.

Applying themes
^^^^^^^^^^^^^^^

The pre-requisite to applying themes in Flatpaks is to have the theme
available to the application in the sandbox, commonly done by packaging
them as theme extensions.

Now the system theme needs to be communicated from host to the sandbox.

On X11, Gtk3 picks up themes from XSettings. Specifically, on GNOME, the
GNOME XSettings daemon ``gsd-xsettings`` reads the DConf values and
converts them into XSettings values. On GNOME, this should work by default
provided ``gsd-xsettings`` is running.

On Wayland, the active theme is exposed via the corresponding Settings
portal. This requires the portal backend for the desktop environment to
be installed.

Once the theme is installed and exposed in the sandbox, depending on the
toolkit/libraries used or the application, it will automatically be
applied.

If the theme is not available via Flatpak extensions or portal
or xsettings support is lacking, Gtk and Qt applications will fallback
to using Adwaita or the default Qt theme.

In this case, either the theme needs to be packaged as an extension
or the application needs to have permission to read the theme and Gtk or
Qt settings from host, usually by giving it filesystem access. This
is undesirable in most cases as it weakens the sandbox and reduces
portability. The desktop environments should provide proper portal or
XSettings daemon support.

If a Gtk theme is not packaged as an extension, an unmaintained extension
can be created for it. Please see
:ref:`extension:Creating an unmaintained Gtk theme extension`.

Appearance Settings
^^^^^^^^^^^^^^^^^^^

Appearance settings such as the Freedesktop color-scheme preference are
also exposed similarly via the respective Settings portal. The
application needs to support reading it and the proper portal backends
are needed to be installed for this to work.

Fonts
------

Flatpak exposes fonts from host to the sandbox and the runtime ships
the default fontconfig from upstream.

Fonts from ``/usr/share/fonts`` are exposed in ``/run/host/fonts``,
``/usr/local/share/fonts`` is exposed in ``/run/host/local-fonts`` and
``$XDG_DATA_HOME/fonts`` in ``/run/host/user-fonts``. 

``~/.fonts`` is a legacy path, and should not be used.

Fontconfig config files from host are not exposed.
