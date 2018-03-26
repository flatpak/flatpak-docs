Python
======

Python applications that use supported build systems like Meson, CMake, or Autotools can be built using the standard method. However, many Python applications use custom install scripts or are expected to be installed through Setuptools and ``pip``.

For these cases, ``flatpak-builder`` provides the ``simple`` buildsystem. Rather than automating the build process, ``simple`` accepts a ``build-commands`` array of strings, which are executed in sequence.

For example, the following JSON makes building the popular requests module rather straightforward:

.. code-block:: json

    {
      "name": "requests",
      "buildsystem": "simple",
      "build-commands": [
        "pip3 install --prefix=/app --no-deps ."
      ],
      "sources": [
        {
          "type": "archive",
          "url": "https://files.pythonhosted.org/packages/source/r/requests/requests-2.18.4.tar.gz",
          "sha256": "9c443e7324ba5b85070c4a818ade28bfabedf16ea10206da1132edaa6dda237e"
        }
      ]
    }

Here, ``build-commands`` is an array containing the commands required to build and install the module. As can be seen, in this case ``pip`` is run to do this. Here, the ``--prefix=/app`` option is important, because otherwise ``pip`` would try to install the module under ``/usr/`` which, because
``/usr/`` is mounted read-only inside the sandbox, would fail.

Note that ``--no-deps`` is only used for the purpose of the example - since the requests module has its own dependencies, the build would fail. If multiple dependencies are required, it is better to install them using the method in the next section, instead.

Building multiple python dependencies
-------------------------------------

Even though the example above installs, it won't actually work. This is because the requests module has a number of dependencies that haven't been installed:

-  certifi
-  chardet
-  idna
-  urllib3

Four dependencies aren't very many, and could be installed these using the ``simple`` method described above. However, anything more complex than this would quickly become tedious.

For these cases, `flatpak-pip-generator <https://github.com/flatpak/flatpak-builder-tools/tree/master/pip>`_ can be used to generate the necessary manifest JSON. This is a Python script that takes a package name and uses ``pip`` to identify its dependencies, along with their tarball URLs and hashes.

Using ``flatpak-pip-generator`` is as simple as running::

    $ python3 flatpak-pip-generator requests

This will output a file called ``python3-requests.json``, containing the necessary manifest JSON, which can then be included in your application's manifest file.
