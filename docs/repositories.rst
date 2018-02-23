Repositories
============

Flatpak repositories are the primary mechanism for publishing applications, so that they can be installed by users.

Some aspects of repositories are addressed by other sections of the documentation. Basic commands for adding, removing and inspecting repositories can be found in the :doc:`using-flatpak` section. Additionally, the section on :doc:`flatpak-builder` covers the most common method for adding applications to repositories.

To use a repository to publish an application, it is possible to either host your own (covered in the next section, :doc:`hosting-a-repository`) or use `Flathub <http://flathub.org>`_, the primary publishing and hosting service for Flatpak applications.

Whether you are hosting your own or using Flathub, once your application has been added to a repository, it can be discovered and installed by users through a software center, like GNOME Software or KDE Discover. It can also be searched for and installed through the command line. A third option is to provide a ``.flatpakref`` file which allows the application to be installed through a download link.

.flatpakref files
-----------------

``.flatpakref`` files are simple description files that include information about a Flatpak application. An example::

  [Flatpak Ref]
  Name=fr.free.Homebank
  Branch=stable
  Title=fr.free.Homebank from flathub
  Url=https://dl.flathub.org/repo/
  RuntimeRepo=https://dl.flathub.org/repo/flathub.flatpakrepo
  IsRuntime=false
  GPGKey=mQINBFlD2sABEADsiUZUO...

As can be seen, the file includes the ID of the application and the location of the repository that contains it, as well a link to information about the repository that provides the application's runtime. In short, the ``.flatpakref`` contains all the information needed to install the application.

The ``flatpak install`` command accepts both local and remote ``.flatpakref`` files. For example::

  $ flatpak install https://flathub.org/repo/appstream/fr.free.Homebank.flatpakref

Or, if the same file has been downloaded::

  $ flatpak install fr.free.Homebank.flatpakref

Alternatively, Flatpak-enabled desktops allow opening a ``.flatpakref`` file to install an application, either by clicking a local file or a download link to the file.

.. note::

  ``.flatpakref`` files should include the base64-encoded version of the GPG key that was used to sign the repository. This can be obtained with the following command::

  $ base64 --wrap=0 < key.gpg

Publishing updates
------------------

Flatpak repositories are similar a Git repositories, in that they store every version of an application by keeping a record of the difference between each version. This makes updating efficient, since only the difference (or "delta") between two versions needs to be downloaded when an update is performed.

When a new version of an application is added to a repository, it immediately becomes available to users. Software centers are able to automatically check for and install new versions. Those who are using the command line have to manually run ``flatpak update`` to check for and install new versions of any applications they have installed.
