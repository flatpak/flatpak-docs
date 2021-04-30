Available Runtimes
==================

This page provides information about available Flatpak runtimes. It is
primarily intended as information for application developers and distributors.

There are currently three main runtimes available: Freedesktop, GNOME and
KDE. These are all hosted on `Flathub <https://flathub.org/>`_.

What is mentioned here is just a high level look at the contents. To have up
to date information simply install the runtime and open a shell inside of it
(``flatpak run org.freedesktop.Sdk//19.08``) from there you can look around or
use tools like ``pkg-config --list-all``. In the runtime shell you can also
inspect ``/usr/manifest.json``, which lists the sources used to build it.

Freedesktop
-----------

The Freedesktop runtime is the standard runtime that can be used for any
application and contains a set of essential libraries and services, including
D-Bus, GLib, Gtk3, PulseAudio, X11 and Wayland.

The Freedesktop runtime is maintained `here
<https://gitlab.com/freedesktop-sdk/freedesktop-sdk/>`__ and has a website
`here <https://freedesktop-sdk.io/>`__.

Available Freedesktop runtimes:

==================================================== =====================================
ID                                                   Description
==================================================== =====================================
org.freedesktop.Platform                             Runtime
org.freedesktop.Platform.Locale                      Runtime translations (extension)
org.freedesktop.Platform.VAAPI.Intel{,.i386}         Intel vaapi drivers (extension)
org.freedesktop.Platform.ffmpeg-full                 All ffmpeg codecs (extension)
org.freedesktop.Platform.Compat.{architecture}       32 bits compatible extension
org.freedesktop.Platform.Compat.{architecture}.debug 32 bits compatible extension (debug)
org.freedesktop.Platform.GL{,32}.default             Mesa drivers (extension)
org.freedesktop.Platform.GL{,32}.mesa-aco            Mesa-ACO variant drivers (extension)
org.freedesktop.Sdk                                  SDK
org.freedesktop.Sdk.Debug                            SDK debug information (extension)
org.freedesktop.Sdk.Locale                           SDK translations (extension)
org.freedesktop.Sdk.Docs                             SDK documentation (extension)
org.freedesktop.Sdk.Extension.rust-stable            SDK Rust language support (extension)
==================================================== =====================================

GNOME
-----

The GNOME runtime is appropriate for any application that uses the GNOME
platform. It is based on the Freedesktop runtime and adds the GNOME platform,
including:

* Clutter
* Gjs
* GObject Introspection
* GStreamer
* GVFS
* Libnotify
* Libsecret
* LibSoup
* PyGObject
* Vala
* WebKitGTK

The GNOME runtime is maintained `here
<https://gitlab.gnome.org/GNOME/gnome-build-meta>`__.

Available GNOME runtimes:

=========================  =================================
ID                         Description
=========================  =================================
org.gnome.Platform         Runtime
org.gnome.Platform.Locale  Runtime translations (extension)
org.gnome.Sdk              SDK
org.gnome.Sdk.Debug        SDK debug information (extension)
org.gnome.Sdk.Locale       SDK translations (extension)
org.gnome.Sdk.Docs         SDK documentation (extension)
=========================  =================================

KDE
---

The KDE runtime is also based on the Freedesktop runtime and adds Qt and KDE
Frameworks. It is appropriate for any application that makes use of the KDE
platform and most Qt-based applications.

The KDE runtime is maintained `here
<https://invent.kde.org/packaging/flatpak-kde-runtime>`__.

Available KDE runtimes:

=======================  =================================
ID                       Description
=======================  =================================
org.kde.Platform         Runtime
org.kde.Platform.Locale  Runtime translations (extension)
org.kde.Sdk              SDK
org.kde.Sdk.Debug        SDK debug information (extension)
org.kde.Sdk.Locale       SDK translations (extension)
org.kde.Sdk.Docs         SDK documentation (extension)
=======================  =================================
