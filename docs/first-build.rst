Building Your First Flatpak
===========================

This tutorial provides a quick, step-by-step example of how to build a simple application as a Flatpak. It also takes the opportunity to explain a few key concepts.

In order to complete this tutorial, you should have followed the `setup guide on flatpak.org <http://flatpak.org/setup/>`_. You also need to have installed ``flatpak-builder``, which is usually available from the same repository as the ``flatpak`` package.

1. Create a manifest
--------------------

The input to ``flatpak-builder`` is a JSON file that describes the parameters for building the application. This is called the manifest. The following example is the manifest for the GNOME Dictionary application:

.. code-block:: json

  {
    "app-id": "org.gnome.Dictionary",
    "runtime": "org.gnome.Platform",
    "runtime-version": "3.26",
    "sdk": "org.gnome.Sdk",
    "command": "gnome-dictionary",
    "finish-args": [
       "--socket=x11",
       "--share=network"
    ],
    "modules": [
      {
        "name": "gnome-dictionary",
        "sources": [
          {
            "type": "archive",
            "url": "https://download.gnome.org/sources/gnome-dictionary/3.26/gnome-dictionary-3.26.0.tar.xz",
            "sha256": "387ff8fbb8091448453fd26dcf0b10053601c662e59581097bc0b54ced52e9ef"
          }
        ]
      }
    ]
  }

As can be seen, this manifest includes basic information about the application, including:

- ``app-id`` - the application ID
- ``runtime`` - the runtime, which provides the environment that the application will run in, as well as basic dependencies it can use
- ``sdk`` - the Software Development Kit, which is a version of the runtime with development files and headers; this is required to build the app
- ``command`` - the command that is used to run the application
- ``finish-args`` - configuration options which give the application access to resources outside of its sandbox; in this case, the application is being given network access and access to the X11 display server

The next part of the manifest is the ``modules`` list. This describes each of the modules that are to be built as part of the build process. One of these modules is always the application. Others can be libraries and other resources that are bundled as part of the application.

Module sources can be of several types, including ``.tar`` or ``.zip`` archives, Git or Bzr repositories. In this case, there is only one module, which is a ``.tar`` file which will be downloaded and built.

The modules section of the Dictionary manifest is short, because only one module is built: the application itself.

To create a manifest for the Dictionary, create a file called ``org.gnome.Dictionary.json`` and paste the JSON from above into it.

2. Run the build
----------------

To use the manifest to build the Dictionary application, run the following command::

  $ flatpak-builder --repo=tutorial-repo dictionary org.gnome.Dictionary.json

This will:

* Create a new directory called dictionary
* Download and verify the Dictionary source code
* Build and install the source code, inside the SDK rather than the host system
* Finish the build, by setting permissions (in this case giving access to X11 and the network)
* Create a new repository called repo (if it doesn't exist) and export the resulting build into it

``flatpak-builder`` will also do some other useful things, like creating a separately installable debug runtime (called ``org.gnome.Dictionary.Debug`` in this case) and a separately installable translation runtime (called ``org.gnome.Dictionary.Locale``).

3. Add the new repository
-------------------------

To test the application that has been built, you need to add the new repository that has been created::

  $ flatpak --user remote-add --no-gpg-verify --if-not-exists tutorial-repo tutorial-repo

4. Install the application
--------------------------

The next step is to install the Dictionary application from the repository. To do this, run::

  $ flatpak --user install tutorial-repo org.gnome.Dictionary

To check that the application has been successfully installed, you can compare the sha256 commit of the installed app with the commit ID that was printed by ``flatpak-builder``::

  $ flatpak info org.gnome.Dictionary
  $ flatpak info org.gnome.Dictionary.Locale

5. Run the application
----------------------

Finally, you can run the application that you've built::

  $ flatpak run org.gnome.Dictionary

The rest of the documentation provides a complete guide to using ``flatpak-builder``. If you are new to Flatpak, it is recommended to start with the :doc:`introduction`.
