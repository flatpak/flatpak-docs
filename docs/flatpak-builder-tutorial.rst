Tutorial
========

To try ``flatpak-builder`` yourself, create a file called ``org.gnome.Dictionary.json`` and paste the Dictionary manifest JSON from :doc:`manifests` into it. Then run the following command::

  $ flatpak-builder --repo=tutorial-repo dictionary org.gnome.Dictionary.json

This will:

* Create a new directory called dictionary
* Download and verify the Dictionary source code
* Build and install the source code, using the SDK rather than the host system
* Finish the build, by setting permissions (in this case giving access to X and the network)
* Create a new repository called repo (if it doesn't exist) and export the resulting build into it

``flatpak-builder`` will also do some other useful things, like creating a separately installable debug runtime (called ``org.gnome.Dictionary.Debug`` in this case) and a separately installable translation runtime (called ``org.gnome.Dictionary.Locale``).

Then you can add the new repository and install the Dictionary application from it. To do this, run::

  $ flatpak --user remote-add --no-gpg-verify --if-not-exists tutorial-repo tutorial-repo
  $ flatpak --user install tutorial-repo org.gnome.Dictionary

To check that the application has been successfully install, you can compare the sha256 commit of the installed app with the commit ID that was printed by flatpak-builder::

  $ flatpak info org.gnome.Dictionary
  $ flatpak info org.gnome.Dictionary.Locale

And finally, you can run the new version of the Dictionary app::

  $ flatpak run org.gnome.Dictionary
