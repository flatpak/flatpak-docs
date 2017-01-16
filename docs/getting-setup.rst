Getting Setup
=============

Getting setup to build Flatpaks is quick and easy. First, it is necessary to have the ``flatpak`` and ``flatpak-builder`` packages installed on your system. These are available for most distributions, and the Flatpak website provides `details on how to get them <http://flatpak.org/getting.html>`_.

Once flatpak has been installed, it is necessary to pick a runtime and install the SDK and install it.

Installing an SDK
-----------------

An SDK is a special type of runtime that is used to build applcations. Typically, an SDK is paired with a runtime that will be used by the application at runtime. For example the GNOME 3.22 SDK is used to build applications that use the GNOME 3.22 runtime. 

The Flatpak website provides a `list of the available runtimes <http://flatpak.org/runtimes.html>`_. Once you have decided which one to use, getting setup is just a matter of installing it and its SDK.

The examples in the rest of the Flatpak documentation uses the GNOME 3.22 runtime and SDK. If you haven't installed these already, download the repository GPG key and then add the repository that contains the runtime and SDK::

  $ flatpak remote-add --from gnome https://sdk.gnome.org/gnome.flatpakrepo

You can now download and install the runtime and SDK::

  $ flatpak install gnome org.gnome.Platform//3.22 org.gnome.Sdk//3.22

This same procedure can be used to install any other runtime and SDK.

Taking a look around
--------------------

If this is the first time you've used Flatpak, this might be a good time to try installing an application and having a look 'under the hood'. To do this, you need to add a repository that contains applications. We can do this using the gnome-apps repository to install gedit::

  $ flatpak remote-add --from gnome-apps https://sdk.gnome.org/gnome-apps.flatpakrepo
  $ flatpak install gnome-apps org.gnome.gedit

You can now use the following command to get a shell in the 'devel sandbox'::

  $ flatpak run --devel --command=bash org.gnome.gedit

This gives you an environment which has the application bundle mounted in ``/app``, and the SDK it was built against mounted in ``/usr``. You can explore these two directories to see what a typical flatpak looks like, as well as what is included in the SDK.
