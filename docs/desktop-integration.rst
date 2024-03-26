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
`search provider <https://developer-old.gnome.org/SearchProvider/>`_. This allows
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

Flatpak applications cannot directly use the system theme. This happens because flatpaks do not have the ability to use data files or libraries in ``/usr`` (where system themes are typically located). The solution to this was to package themes as Flatpaks, as relying upon the host to have the correct version for every flatpak defeats the portability benefits it provides. These themes are provided as `extensions <https://github.com/flatpak/flatpak/wiki/Extensions>`_, to the Freedesktop runtime when the extension point is Gtk, and to the KDE runtime when the extension point is Qt.

The theming system requires Flatpak 0.8.4+ and applications using up to date ``org.gnome.Platform`` 3.24+, or ``org.freedesktop.Platform`` 1.6+, or ``org.kde.Platform`` 5.9+.

Installing themes
^^^^^^^^^^^^^^^^^

Instructions for Gtk
````````````````````

The current Gtk themes are packaged in the `flathub <https://flathub.org/>`_ repository which you can add (if it's not already added) by running::

  $ flatpak remote-add flathub https://flathub.org/repo/flathub.flatpakrepo

To see a list of currently packaged themes you can use the command ``flatpak search gtk3theme`` (available since Flatpak version 0.10.1). In case you use an older version of Flatpak than that, you can use the command ``flatpak remote-ls flathub | grep org.gtk.Gtk3theme``. The difference in output between these two commands is that the first prints the application ID, the remote from which the theme comes and the theme's description, while the second prints only the full name of the theme's flatpak package.

You can install themes with the command ``flatpak install flathub org.gtk.Gtk3theme.Foo``, replacing ``Foo`` with the name of the desired theme.

Instructions for Qt
```````````````````

For the Qt theming to work, the flatpak packages kstyle and platformtheme must be installed. These are packed in the kdeapps repository which you can add by running::

  $ flatpak remote-add kdeapps https://distribute.kde.org/kdeapps.flatpakrepo

Afterwards the two packages can be installed with the following commands::

  $ flatpak install kdeapps org.kde.KStyle.Adwaita//5.9
  $ flatpak install kdeapps org.kde.PlatformTheme.QGnomePlatform//5.9

Applying themes
^^^^^^^^^^^^^^^

Introduction
```````````````````

There is no ideal way to specify the theme Flatpak applications use. The applications will try to match the system theme currently being used, if it corresponds to any of the Flatpak themes installed, and will fall back to Adwaita (if they use Gtk2 or Gtk3) or the default Qt theme (if they use Qt) if a corresponding theme isn't detected.

As of Flatpak 0.10.1, the Flatpak system can detect whether the system themes available correspond to any Flatpak themes available in the repositories, and, if so, will automatically install found themes at update time based upon the ``gtk-theme`` Dconf key. This key however can contain only one value, the one of the currently being used theme, which means that the Flatpak versions of matching themes that aren't currently being used aren't installed until those themes are enabled. If none of the corresponding system themes are currently being used, the applications will fall back to Adwaita or the default Qt theme.

On X11, Gtk3 picks up the themes via XSettings. Specifically, the GNOME XSettings daemon ``gsd-xsettings`` reads the DConf values and converts them into the XSettings values. For this to work, you need an xsettings daemon that is correctly configured. Gtk3 on Wayland picks up themes directly via Dconf. For this to work, you can either use KDE (with ``kde-gtk-config`` > 5.11.95), GNOME, which works out of the box, or manually configure the dconf keys under ``/org/gnome/desktop/interface/``. For the DConf option to work on Wayland the application must also be configured to have DConf access.

Steps with Flatseal
```````````````````

Summarized steps for users to set a default theme for either all Flatpak applications or selected Flatpak applications: Navigate to **All Applications** horizontal tab —> **Filesystem** group —> **Other files** subgroup; Add the appropriate variable(s). Redo the same for **Environment** group. In the future, most future Flatpak applications you open will use the new theme.

This screenshot shows the Flatseal configuration at https://i.postimg.cc/9M9jwVKC/flatseal-theme.png

The reason why Flatpak does not automatically use the user's default operating system (OS) theme is that, by default, all Flatpak applications are almost fully independent of the OS. Think of two independent devices inside your device. This is much stronger security. The downside is that sometimes, you need both your devices to communicate with each other. So that they can use shared settings or anything else to your liking. Such as themes and icons.

Flatseal resolve this challenge. Flatseal is a free permission manager for any Flatpak applications.

------------

Below is are the same steps as above. But with details for users not familiar with Flatseal.

1. Keep in mind that the below steps will automatically set a default theme to your liking for most of all Flatpak applications. Including both the one already installed and those installed in the future. Be aware that all your Flatpak packages will be allowed to access all the theme settings on your operating system (OS). If you’re ok with this, find your next action below.

2. Install this free Flatseal Flatpak application at https://flathub.org/fr/apps/com.github.tchx84.Flatseal

3. Using Flatseal, navigate to **All Applications** horizontal tab —> **Filesystem** group —> **Other files** subgroup

4. In this **Other** files group, on the right side, click on the small plus (**+**) button. Add the appropriate **GTK_THEME** Filesystem variable(s) for any theme to your liking. For example, for **Adwaita-dark**, add those two:

.. code-block:: yaml

   ~/.themes

.. code-block:: yaml

   ~/.icons

5. Still using Flatseal, navigate to **All Applications** horizontal tab —> **Environment** group —> Variables subgroup

6. In this **Variables** group, on the right side, click on the small plus (**+**) button. Add the appropriate **ICON_THEME** Environment variable(s) for any theme to your liking. For example, for **Adwaita-dark**, add those two:

.. code-block:: yaml

   GTK_THEME=Adwaita-dark

.. code-block:: yaml

   ICON_THEME=Adwaita-dark

7. This screenshot https://i.postimg.cc/9M9jwVKC/flatseal-theme.png shows the end result for Adwaita-dark theme. Including both the theme and icons variables. See the screenshot above for an example of the final configuration.

8. Flatseal conveniently and automatically save any added variable or change to variables. In other words, you do not need to click on any Save button. Close Flatseal.

9. Open Flatseal. Flatseal now automatically open using the new theme you set. Congratulation. You have successfully set a new theme for most Flatpak applications.

10. In the future, most future Flatpak applications you open will automatically and permanently use the new theme of your choosing. Easy.

11. Optionally, if you need to set a different theme per Flatpak application, using Flatseal, navigate to **<APP NAME>** horizontal tab. Then redo that same steps as above. But only for this one app. This screenshot shows an example of a final result with the **Calculator** Flatpak Application.

Attribution and thanks to Sreenath for his publication about Flatseal at [https://itsfoss.com/flatpak-app-apply-theme/ 11](https://itsfoss.com/flatpak-app-apply-theme/)

Other notes on theming
^^^^^^^^^^^^^^^^^^^^^^

In regards to icon themes, since Flatpak 0.8.8 the host icons are exposed to the guest, so that there is usually no need for the presence of Flatpak icon themes.

If you use the *Global Dark Theme* option (removed in GNOME-Tweaks 3.28)  in ``gnome-tweak-tool`` it will not work as that simply writes to ``settings.ini`` which isn’t available in the sandbox. Use dark versions of themes instead if they exist.
