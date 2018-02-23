Hosting a repository
====================

The section on :doc:`flatpak-builder` describes how to generate repositories ``flatpak-builder``. The resulting OSTree repository can be hosted on a web server for consumption by users.

Important details
-----------------

OSTree repositories use archive-z2, meaning that they contain a single file for each file in the application. This means that pull operations involve a lot of HTTP requests. Since new requests can be slow, it is important to enable HTTP keep-alive on the web server that is hosting your repository.

OSTree's static deltas feature uses single files that contain all the data needed to update between two revisions (or update from nothing to a revision). Creating such deltas consumes more space on the server, but reduces download sizes. This can be done with the ``build-update-repo --generate-static-deltas`` option.

GPG signatures
--------------

OSTree uses GPG to verify the identity of repositories. This requires that every commit to a repository uses a GPG signature, as well as when repository summary files are modified.

To do this, a GPG key needs to be passed to ``flatpak-builder`` if it is being used to modify or create a repository. (If you don't already have a key, `it is easy to generate one <https://help.github.com/articles/generating-a-new-gpg-key/>`_.) For example::

  $ flatpak build-export --gpg-sign=KEYID --gpg-homedir=PATH REPOSITORY DIRECTORY

Here ``--gpg-homedir`` is optional, and allows specifying the home directory of the key to be used.

Though it generally isn't recommended, it is possible to disable GPG verification of OSTree repositories. To do this, the ``--no-gpg-verify`` option can be used when a remote is added. GPG verification can also be disabled on an existing remote using ``flatpak remote-modify``.

Note that it is necessary to become root in order to update a remote that does not have GPG verification enabled.

.flatpakrepo files
------------------

``.flatpakrepo`` files are a convenient way to let users add a repository. These are simple description files which contain information about the repository. For example, the Flathub repo file looks like::

  [Flatpak Repo]
  Title=Flathub
  Url=https://dl.flathub.org/repo/
  Homepage=https://flathub.org/
  Comment=Central repository of Flatpak applications
  Description=Central repository of Flatpak applications
  Icon=https://dl.flathub.org/repo/logo.svg
  GPGKey=mQINBFlD2sABEADsiUZUO...

Here you can see that the repo file contains descriptive metadata, such as the repository name, description, icon and website. The file also contains information that is needed to add the repository, including a download URL and the repository's GPG key.

``.flatpakrepo`` files can be used to add a repository from the command line. For example, the command to add Flathub using its repo file is::

  $ flatpak remote-add --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo

The command line isn't the only way to add a repository using a ``.flatpakrepo`` file - on desktops that support Flatpak, it is just a matter of clicking the repository file or a download link that points to it.

.. note::

  ``.flatpakrepo`` files should include the base64-encoded version of the GPG key that was used to sign the repository. This can be obtained with the following command::

  $ base64 --wrap=0 < key.gpg
