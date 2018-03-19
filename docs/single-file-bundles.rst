Single-file bundles
===================

Hosting a repository is the preferred way to distribute an application, since repositories allow applications to be updated. However, sometimes it can be appropriate to use a single-file bundle. These can be used to provide a direct download of the application, to distribute applications using removable media, or to send them as email attachments.

Flatpak allows single file bundles to be created with the ``build-bundle`` and ``build-import-bundle`` commands, which allow an application in a repository to be converted into a bundle and back again::

  $ flatpak build-bundle [OPTION...] LOCATION FILENAME NAME [BRANCH]
  $ flatpak build-import-bundle [OPTION...] LOCATION FILENAME

For example, to create a bundle named `dictionary.flatpak` containing the GNOME dictionary app from the repository at ~/repositories/apps, run::

  $ flatpak build-bundle ~/repositories/apps dictionary.flatpak org.gnome.Dictionary

To import the bundle into a repository on another machine, run::

  $ flatpak build-import-bundle ~/my-apps dictionary.flatpak

Alternatively, bundles can also be installed directly without importing them::

  $ flatpak install dictionary.flatpak
