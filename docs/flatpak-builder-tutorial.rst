Tutorial
========

This tutorial provides a sample set of step that you can use to try ``flatpak-builder`` yourself. In it, you will learn how to use ``flatpak-builder`` to build the GNOME Dictionary applicaiton.

1. Create a manifest
--------------------

To create a manifest for the application, create a file called ``org.gnome.Dictionary.json`` and paste the Dictionary manifest JSON from :doc:`manifests` into it.

2. Run the build
----------------

To use the manifest to build the Dictionary application, run the following command::

  $ flatpak-builder --repo=tutorial-repo dictionary org.gnome.Dictionary.json

This will:

* Create a new directory called dictionary
* Download and verify the Dictionary source code
* Build and install the source code, using the SDK rather than the host system
* Finish the build, by setting permissions (in this case giving access to X11 and the network)
* Create a new repository called tutorial-repo (if it doesn't exist) and export the resulting build into it

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
