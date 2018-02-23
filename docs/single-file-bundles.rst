Single-file bundles
===================

Hosting a repository is the preferred way to distribute an application, but sometimes a single-file bundle that you can make available from a website or send as an email attachment is more convenient. Flatpak supports this with the ``build-bundle`` and ``build-import-bundle`` commands to convert an application in a repository to a bundle and back::

  $ flatpak build-bundle [OPTION...] LOCATION FILENAME NAME [BRANCH]
  $ flatpak build-import-bundle [OPTION...] LOCATION FILENAME

For example, to create a bundle named `dictionary.flatpak` containing the GNOME dictionary app from the repository at ~/repositories/apps, run::

  $ flatpak build-bundle ~/repositories/apps dictionary.flatpak org.gnome.Dictionary

To import the bundle into a repository on another machine, run::

  $ flatpak build-import-bundle ~/my-apps dictionary.flatpak

Note that bundles have some drawbacks, compared to repositories. For example, distributing updates is much more convenient with a hosted repository, since users can just run ``flatpak update``.
