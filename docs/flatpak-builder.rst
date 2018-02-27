Flatpak Builder
===============

Flatpaks are built using the ``flatpak-builder`` tool. This allows a series of modules to be built into a single application bundle. These modules can include libraries and dependencies in addition to the application itself.

Modules can be built with a variety of build systems, including `autotools <https://www.gnu.org/software/automake/manual/html_node/Autotools-Introduction.html>`_, `cmake <https://cmake.org/>`_, `cmake-ninja <https://cmake.org/cmake/help/v3.0/generator/Ninja.html>`_, `meson <http://mesonbuild.com/>`_, and the so called `Build API <https://github.com/cgwalters/build-api/>`_. A "simple" build method is also available, which allows a series of commands to be specified.

Exporting
---------

The result of the build process can be exported to a repository or automatically installed locally.

Exporting to a repository
`````````````````````````

The ``--repo`` option allows a repository to be specified, which the resulting application will be added to. This takes the format::

 $ flatpak-builder --repo=<repository-destination> application.id.json


.. note::

  By default, flatpak-builder splits off translations and debug information into separate `.Locale` and `.Debug` extensions. These extensions are automatically exported into a repository along with the application.


Installing builds directly
``````````````````````````

Instead of exporting to a repository, the application bundle that is produced by ``flatpak-builder`` can be automatically installed locally::

  $ flatpak-builder --install application.id.json

Signing
-------

Every commit to a Flatpak repository should be signed with a GPG signature. If ``flatpak-builder`` is being used to modify or create a repository, a GPG key should therefore be passed to it. This can be done with the ``--gpg-sign`` option, such as::

  $ flatpak-builder --gpg-sign=<key-id> --repo=<repository-destination> application.id.json

The ``--gpg-homedir`` option can also be used to specify the home directory of the key that is being used.

Though it generally isn't recommended, it is possible not to use GPG verification. In this case, the ``--no-gpg-verify`` option should be used when adding the repository. Note that it is necessary to become root in order to update a repository that does not have GPG verification enabled.
