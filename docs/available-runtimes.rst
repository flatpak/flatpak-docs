Available Runtimes
==================

This page provides information about available Flatpak runtimes. It is primarily intended as information for application developers and distributors.

There are currently three main runtimes available: Freedesktop, GNOME and KDE. These are all hosted on `Flathub <https://flathub.org/>`_.

Freedesktop
-----------

The Freedesktop runtime is the standard runtime that can be used for any application and contains a set of essential libraries and services, including D-Bus, GLib, PulseAudio, X11 and Wayland.

Available Freedesktop runtimes:

===============================  =================================
ID                               Description
===============================  =================================
org.freedesktop.Platform         Runtime
org.freedesktop.Platform.Locale  Runtime translations (extension)
org.freedesktop.Sdk              SDK
org.freedesktop.Sdk.Debug        SDK debug information (extension)
org.freedesktop.Sdk.Locale       SDK translations (extension)
org.freedesktop.Sdk.Docs         SDK documentation (extension)
===============================  =================================

GNOME
-----

The GNOME runtime is appropriate for any application that uses the GNOME platform. It is based on the Freedesktop runtime and adds the GNOME platform, including:

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

The KDE runtime is also based on the Freedesktop runtime and adds Qt and KDE Frameworks. It is appropriate for any application that makes use of the KDE platform and most Qt-based applications.

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
