Hosting a repository
====================

The previous sections of this guide describe how to generate repositories using ``build-export`` or ``flatpak-builder``. The resulting OSTree repository can be hosted on a web server for consumption by users.

Important details
-----------------

OSTree repositories use archive-z2, meaning that they contain a single file for each file in the application. This means that pull operations will do a lot of HTTP requests. Since new requests are slow, it is important to enable HTTP keep-alive on the web server.

OSTree supports something called static deltas. These are single files in the repo that contains all the data needed to go between two revisions (or from nothing to a revision). Creating such deltas will take up more space on the server, but will make downloads much faster. This can be done with the ``build-update-repo --generate-static-deltas`` option.

GPG signatures
--------------

OSTree uses GPG to verify the identity of repositories. This requires that every commit to a repository uses a GPG signature, as well as when repository summary files are modified.

To do this, a GPG key needs to be passed to the ``build-update-repo`` and ``build-export`` commands, as well as ``flatpak-builder`` if it is being used to modify or create a repository. (If you don't already have a key, `it is easy to generate one <https://help.github.com/articles/generating-a-new-gpg-key/>`_.) For example::

  $ flatpak build-export --gpg-sign=KEYID --gpg-homedir=PATH REPOSITORY DIRECTORY

Here ``--gpg-homedir`` is optional, and allows specifying the home directory of the key to be used.

Though it generally isn't recommended, it is possible to disable GPG verification of OSTree repositories. To do this, the ``--no-gpg-verify`` option can be used when a remote is added. GPG verification can also be disabled on an existing remote using ``flatpak remote-modify``.

Note that it is necessary to become root in order to update a remote that does not have GPG verification enabled.
