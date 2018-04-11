Electron tutorial
=================

This guide builds a simple electron example application hosted at:

https://github.com/flathub/electron-sample-app

flatpak-builder method
======================

This will guide you through the process of writing a json manifest to
build our example application.

The finished manifest can be found here:

https://github.com/flathub/electron-sample-app/blob/master/flatpak/

If you only want to build it and skip the rest of the guide, download
the contents of the folder (both files are necessary) and run:

::

    flatpak-builder build org.flatpak.electron-sample-app.json --install

If you have added the flathub remote, you may wish to add
``--install-deps-from=flathub`` so that it automatically downloads the
necessary build runtime and base app.

Once that's done, test it with:

::

    flatpak run org.flatpak.electron-sample-app

Minimal permissions
-------------------

At the very least you need:

::

    "--share=ipc",
    "--socket=x11",

Electron does not currently support wayland.

If your application plays sound:

::

    "--socket=pulseaudio",

If your application accesses the network:

::

    "--share=network"

Using the electron base app
---------------------------

Electron requires some legacy dependencies like gtk2 that the
freedesktop runtime doesn't include. It would be possible to build these
as modules manually, but because these dependencies are the same for
every electron app, a base app has been created that includes them and
allows installing nodejs and electron without any further issue.

In order to be used, it must be included in the manifest:

::

    "base": "io.atom.electron.BaseApp",
    "base-version": "stable",

Using the base app also has other advantages:

- Deduplication, by ensuring they are built in the same way, it allows ostree to deduplicate the dependencies. This way users with several electron apps installed won't need to waste hard drive storage on them.

- Decreased build times, as it only needs to be built once.

Building nodejs
---------------

The electron base app does not include nodejs (only its dependencies),
it is necessary to build it as a module. This tutorial builds 8.11.1
because it works with most projects at the time of writing, but make
sure to use whichever version is best for your project.

::

    {
        "name": "nodejs",
        "cleanup": [
            "/include",
            "/share",
            "/app/lib/node_modules/npm/changelogs",
            "/app/lib/node_modules/npm/doc",
            "/app/lib/node_modules/npm/html",
            "/app/lib/node_modules/npm/man",
            "/app/lib/node_modules/npm/scripts"
        ],
        "sources": [
            {
                "type": "archive",
                "url": "https://nodejs.org/dist/v8.11.1/node-v8.11.1.tar.xz",
                "sha256": "40a6eb51ea37fafcf0cfb58786b15b99152bec672cccf861c14d1cca0ad4758a"
            }
        ]
    }

Note that the cleanup isn't strictly necessary, but removing
documentation helps reduce final disk size of the bundle.

Getting your npm sources
------------------------

Because npm won't be able to access the network from inside the sandbox,
dependencies must be downloaded as sources and installed using
``--offline``.

The nature of electron development makes doing this manually
impractical, as even the simplest applications depend on multiple dozen
different packages. A `python
script <https://github.com/flatpak/flatpak-builder-tools/tree/master/npm>`__
has been developed to scrape the packages from npm and include them in
your sources.

Using it requires a ``package-lock.json`` file. If you aren't familiar
with nodejs, this file contains information about the versions of
different packages that your application depends on and can be generated
by running ``npm install`` on your application's root directory.

Once you have your ``package-lock.json`` file, the sources can be
generated:

::

    python3 flatpak-npm-generator.py package-lock.json

This will output a file called ``generated-sources.json``, its contents
can be directly embedded in your manifest, but it is usually so large
that keeping it separate and linking to it is more convenient.

Building your application
-------------------------

We'll create a new module to build the application. It can be any name,
but keep in mind that the module's name is also the directory name used
while building as in ``/run/build/$name``.

::

    "name": "electron-sample-app",

We need to add an env variable so that electron-download will find the
cached binaries inside the sandbox:

::

    "build-options" : {
        "env": {
            "electron_config_cache": "/run/build/electron-sample-app/npm-cache"
        }
    },

We add our sources:

::

    "sources": [

Our example application:

::

        {
            "type": "archive",
            "url": "https://github.com/flathub/electron-sample-app/archive/1.0.0.tar.gz",
            "sha256": "221582f14afbe9d723ee1b1737800dcce843a776ebfe8edb5c1e7a1d0d36e7f5",
            "dest": "main"
        },

``generated-sources.json`` is the output of running
``flatpak-npm-generator.py`` on ``package-lock.json``, it contains all
our dependencies. Note that for this to work the file must be in the
same folder:

::

        "generated-sources.json",

We also create a simple script so that flatpak will know how to run the
app. It could be named anything else, but note that it has to be named
in the manifest's ``"command":`` in order to work:

::

        {
            "type": "script",
            "dest-filename": "run.sh",
            "commands": [ "npm start --prefix=/app/main" ]
        }
    ],

The simple buildsystem doesn't feature any automation, but executes the
commands passed in ``build-commands``:

::

    "buildsystem": "simple",

With these we build the app, copy it to ``/app/main/`` and install our
``run.sh`` script to ``/app/bin/`` so that it will be on the ``$PATH``:

::

    "build-commands": [
        "npm install --prefix=main --offline --cache=/run/build/electron-sample-app/npm-cache/",
        "mkdir -p /app/main /app/bin",
        "cp -ra main/* /app/main/",
        "install run.sh /app/bin/"
    ]
