Building python modules
-----------------------

Python applications that use supported buildsystems such as meson,
cmake, or autotools can be built in the same way as regular c
applications. However, many python apps and dependencies use custom
install scripts or are expected to be installed through setuptools and
pip.

For these cases, ``flatpak-builder`` provides the ``simple``
buildsystem. Rather than trying to automate the process like the other
buildsystems, ``simple`` takes a ``build-commands`` array of strings and
only executes the commands given there.

For example, this makes building the popular requests module rather
straightforward:

::

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

``name`` is the name of the module, in practice this also means the name
folder in which your module will be built.

``build-commands`` is an array with the commands you need to build and
install your module. In this case we are running pip to do it, but the
parameters are important. ``--prefix=/app`` is necessary, because
otherwise pip would try to install it under ``/usr/`` and (because
``/usr/`` is mounted read-only inside the sandbox) it would fail.

``--no-deps`` is also relevant, flatpak-builder downloads all
``sources`` before it starts building, and doesn't allow network access
during once a build starts. This means that any attempts to download any
further dependencies past that point would fail. It is used here for
illustrative purposes, but it can be detrimental because it will make
pip quiet about real errors. If you must install multiple dependencies,
it is better to do it in one go using the method in the next section.

Building multiple python dependencies
-------------------------------------

You'll notice that, even though it installs fine, it doesn't actually
work. This is because requests has a number of dependencies that we've
neglected to install:

-  certifi
-  chardet
-  idna
-  urllib3

Four dependencies aren't too much work, we could install all these using
the same method and it would work just fine. However, anything more
complex than this would quickly become tedious.

For these cases, we can use
`flatpak-pip-generator <https://github.com/flatpak/flatpak-builder-tools/tree/master/pip>`_,
this is a python script that takes a package name and uses pip to track
its dependencies, tarball urls and hashes.

Using it is as simple as:

::

    $ python3 flatpak-pip-generator requests

This will output a file called ``python3-requests.json`` containing json
data that can be directly included among your manifest's modules.
