Getting Setup
=============

Install flatpak and flatpak-builder
-----------------------------------

First, it is necessary to have the ``flatpak`` and ``flatpak-builder`` packages installed on your system. The Flatpak website provides `details on how to install flatpak <http://flatpak.org/getting.html>`_. ``flatpak-builder`` is typically found as a package from the same source as ``flatpak`` itself.

Add the flathub repository
--------------------------

Flathub is the main Flatpak repository and contains the runtimes and SDKs that will be needed to run and build apps. If you haven't added it already, it can be setup with the following command::

  $ flatpak remote-add --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo

Install an SDK
--------------

An SDK is a special type of runtime that is used to build applications. Typically, an SDK is paired with a runtime that will be used by the application at runtime. For example the GNOME 3.26 SDK is used to build applications that use the GNOME 3.26 runtime.

Many of the examples in the rest of the Flatpak documentation use the GNOME 3.26 runtime and SDK. To install them, run::

  $ flatpak install flathub org.gnome.Platform//3.26 org.gnome.Sdk//3.26

This same procedure can be used to install any other runtime and SDK.

Taking a look around
--------------------

If this is the first time you've used Flatpak, it is a good time to try installing an application and having a look 'under the hood'. To do this, try installing gedit::

  $ flatpak install flathub org.gnome.gedit

You can now use the following command to get a shell in the 'devel sandbox'::

  $ flatpak run --devel --command=bash org.gnome.gedit

This gives you an environment which has the application bundle mounted in ``/app``, and the SDK it was built against mounted in ``/usr``. You can explore these two directories to see what a typical flatpak looks like, as well as what is included in the SDK.
