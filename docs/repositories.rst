Repositories
============

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
  RuntimeRepo=https://sdk.gnome.org/gnome.flatpakrepo

Note that the GPGKey key in these files contains the base64-encoded GPG key, which you can get with the following command::

  $ base64 --wrap=0 < foo.gpg
