Setup
=====

Install flatpak and flatpak-builder
-----------------------------------

First, it is necessary to have the ``flatpak`` and ``flatpak-builder`` packages installed on your system. The Flatpak website provides `details on how to install flatpak <http://flatpak.org/getting.html>`_. ``flatpak-builder`` is typically found as a package from the same source as ``flatpak`` itself.

Add the flathub repository
--------------------------

Flathub is the main Flatpak repository and contains the runtimes and SDKs that will be needed to run and build apps. If you haven't added it already, it can be setup with the following command::

  $ flatpak remote-add --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo
