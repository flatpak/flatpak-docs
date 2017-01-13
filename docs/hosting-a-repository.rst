Hosting a repository
====================

While it is relatively simple to host a flatpak repository, there are some important details to be aware of.

archive-z2 repositories contain a single file for each file in the application. This means that pull operations will do a lot of HTTP requests. Since new requests are slow, it is important to enable HTTP keep-alive on the web server.

OSTree supports something called static deltas. These are single files in the repo that contains all the data needed to go between two revisions (or from nothing to a revision). Creating such deltas will take up more space on the server, but will make downloads much faster. This can be done with the ``build-update-repo --generate-static-deltas``.

GPG signatures
--------------

By default OSTree refuses to pull from a remote repository that is not signed. To disable GPG verification, the ``--no-gpg-verify`` option needs to be used when a remote is added. Alternatively, it can be disabled on an existing remote using ``flatpak remote-modify``.

Note that GPG signatures are required for the user to be able to install trusted remotes that can be updated from without needing to be root.

OSTree requires signatures for every commit and on repository summary files. These objects are created by the ``build-update-repo`` and ``build-export`` commands, as well as indirectly by ``flatpak-builder``. A GPG key should therefore be passed to each of these commands, and optionally the GPG home directory to use. For example::

  $ flatpak build-export --gpg-sign=KEYID --gpg-homedir=/some/dir appdir repo

Referring to repositories
-------------------------

A convenient way to point users to the repository containing your application is to provide a ``.flatpakrepo`` file that they can download and install. To install a ``.flatpakrepo`` file manually, use the command::

  $ flatpak remote-add --from foo.flatpakrepo

A typical ``.flatpakrepo`` file looks like this::

  [Flatpak Repo]
  Title=GEdit
  Url=http://sdk.gnome.org/repo-apps/
  GPGKey=mQENBFUUCGcBCAC/K9WeV4xCaKr3...

If your repository contains just a single application, it may be more convenient to use a .flatpakref file instead, which contains enough information to add the repository and install the application at the same time. To install a ``.flatpakref`` file manually, use the command::

  $ flatpak install --from foo.flatpakref

A typical ``.flatpakref`` file looks like this::

  [Flatpak Ref]
  Title=GEdit
  Name=org.gnome.gedit
  Branch=stable
  Url=http://sdk.gnome.org/repo-apps/
  IsRuntime=False
  GPGKey=mQENBFUUCGcBCAC/K9WeV4xCaKr3...

Note that the GPGKey key in these files contains the base64-encoded GPG key, which you can get with the following command::

  $ base64 --wrap=0 < foo.gpg

Single-file bundles
-------------------

Hosting a repository is the preferred way to distribute an application, but sometimes a single-file bundle that you can make available from a website or send as an email attachment is more convenient. Flatpak supports this with the ``build-bundle`` and ``build-import-bundle`` commands to convert an application in a repository to a bundle and back::

  $ flatpak build-bundle [OPTION...] LOCATION FILENAME NAME [BRANCH]
  $ flatpak build-import-bundle [OPTION...] LOCATION FILENAME

For example, to create a bundle named `dictionary.flatpak` containing the GNOME dictionary app from the repository at ~/repositories/apps, run::

  $ flatpak build-bundle ~/repositories/apps dictionary.flatpak org.gnome.Dictionary

To import the bundle into a repository on another machine, run::

  $ flatpak build-import-bundle ~/my-apps dictionary.flatpak

Note that bundles have some drawbacks, compared to repositories. For example, distributing updates is much more convenient with a hosted repository, since users can just run ``flatpak update``.
