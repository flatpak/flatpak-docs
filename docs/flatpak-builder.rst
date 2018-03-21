Flatpak Builder
===============

``flatpak-builder`` has already been introduced in :doc:`first-build`, which gave a basic example of how it can be used. This page provides additional detail on how to use ``flatpak-builder``, including the various command options that are available.

Invocation
----------

The basic format used to invoke ``flatpak-builder`` is::

 $ flatpak-builder <directory> <manifest>

Where ``<directory>`` is the path to the directory that the application will be built into, and ``<manifest>`` is the path to a manifest JSON file.

The contents of ``<directory>`` can be useful for testing and debugging purposes, but is generally treated as an artifact of the build process.

Exporting
---------

``flatpak-builder`` provides two options for exporting an application in order to run it. The first is to export to a repository, from which the application can be run. The second is to automatically install locally.

Exporting to a repository
`````````````````````````

The ``--repo`` option allows a repository to be specified, for the application to be exported to. This takes the format::

 $ flatpak-builder --repo=<repository> <directory> <manifest>

Here, ``<repository>`` is a path to a repository. If no repository exists at the specified location, the repository will be created. If the application is already in the specified repository, ``flatpak-builder`` will add the build as a new version of the existing application.

.. note::

  By default, ``flatpak-builder`` splits off translations and debug information into separate `.Locale` and `.Debug` extensions. These extensions are automatically exported into a repository along with the application.


Installing builds directly
``````````````````````````

Instead of exporting to a repository, the Flatpak that is produced by ``flatpak-builder`` can be automatically installed locally, using the ``--install`` option::

  $ flatpak-builder --install <directory> <manifest>

This approach has the advantage of skipping the separate install step that is needed when exporting to a repository.

Signing
-------

Every commit to a Flatpak repository should be signed with a GPG signature. If ``flatpak-builder`` is being used to modify or create a repository, a GPG key should therefore be passed to it. This can be done with the ``--gpg-sign`` option, such as::

  $ flatpak-builder --gpg-sign=<key> --repo=<repository> <manifest>

Here, ``<key>`` is the ID of the GPG key that is to be used. The ``--gpg-homedir`` option can also be used to specify the home directory of the key that is being used.

Though it generally isn't recommended, it is possible not to use GPG verification. In this case, the ``--no-gpg-verify`` option should be used when adding the repository. Note that it is necessary to become root in order to update a repository that does not have GPG verification enabled.
