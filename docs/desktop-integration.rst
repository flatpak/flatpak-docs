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

Toolkits like GTK and Qt provide transparent support for portals:

.. toctree::
   :maxdepth: 2

   portals-gtk
   portals-qt

If you are not using one of these toolkits, it is possible to access
the portals API directly. See the `Portals API documentation
<https://flatpak.github.io/xdg-desktop-portal/portal-docs.html>`_ for more
information.

Notifications
-------------

A number of toolkits and frameworks provide transparent support for Linux
desktop notifications. This includes Electron, GTK, KDE and QML.

Status icons
------------

Status icons are the same concept as the system tray or the taskbar on Windows,
or menu bar icons on Mac. These are supported on most Linux distributions,
through libappindicator.

A number of Linux distributions don't show status icons. It is still possible
to provide a status icon, and it will be shown in some distributions. However,
in order to ensure compatibility, it is recommended to only use status icons
in a supplementary manner, and not to rely on them as the only mechanism for
providing status information or access to particular features. This includes
"minimize to tray" (or equivalent) functionality.

XEmbed style icons will function with the ``x11`` permission but all other
status icon interfaces require extra permissions to escape the sandbox and
these services are not designed to be robust against untrusted software.

System search
-------------

GNOME-based distributions, like CentOS, Fedora, Red Hat Enterprise Linux and
Ubuntu, provide the option to integrate with system search, by providing a
`search provider <https://developer.gnome.org/SearchProvider/>`_. This allows
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

Applications can rely on system-provided titlebars on Linux, if they don't
want to draw their own window controls.

Window decorations
------------------

If your application uses a dark visual style as well as system-provided window
decorations, the ``GTK_THEME_VARIANT=dark`` X11 window property should be
used, to ensure that window decorations match the rest of the application
window. This can be done by running::

  xprop -f _GTK_THEME_VARIANT 8u -set _GTK_THEME_VARIANT dark

Global menus
------------

If your application uses the built in ``GtkApplication:menu-bar`` or the Qt 5
equivalent they will work as expected from within a sandboxed application.
