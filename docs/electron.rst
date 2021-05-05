Electron
========

Due to the nature of Electron, building Electron applications as Flatpaks
requires a few extra steps compared with other applications. Thankfully,
several tools and resources are available which make this much easier.

This guide provides information on how building Electron applications differs
from other applications. It also includes information on the tooling for
building Electron applications and how to use it.

The guide walks through the `manifest file
<https://github.com/flathub/electron-sample-app/blob/master/flatpak/org.flathub.electron-sample-app.yml>`_
of the `sample Electron Flatpak application
<https://github.com/flathub/electron-sample-app>`_. Before you start,
it is a good idea to take a look at this, either online or by downloading
the application.


Building the sample application
-------------------------------

While it isn't strictly necessary, you might want to try building and running
the sample application yourself.

To get setup for the build, download or clone the sample app from GitHub,
and navigate to the ``/flatpak`` directory in the terminal. You must also
install the Electron base app::

  $ flatpak install flathub io.atom.electron.BaseApp//stable

Then you can run the build::

  $ flatpak-builder build org.flathub.electron-sample-app.yml --install --force-clean --user

Finally, the application can be run with::

  $ flatpak run org.flathub.electron-sample-app

Basic configuration
-------------------

The first part of the sample application's manifest specifies the application's
ID. It also configures the runtime and SDK:

.. code-block:: yaml

  app-id: org.flathub.electron-sample-app
  runtime: org.freedesktop.Platform
  runtime-version: '1.6'
  branch: stable
  sdk: org.freedesktop.Sdk

The Freedesktop runtime is generally the best runtime to use with Electron
applications, since it is the most minimal runtime, and other dependencies
will be specific to Electron itself.

The Electron base app
---------------------

Next, the manifest specifies that the Electron base app should be used, by
specifying the ``base`` and ``base-version`` properties in the application
manifest::

  base: io.atom.electron.BaseApp
  base-version: stable

Base apps are described in :doc:`dependencies`.  Using the Electron base
app is much faster and more convenient than manually building Electron and its
dependencies. It also has the advantage of reducing the amount of duplication
on users' machines, since it means that Electron is only saved once on disk.

Note that this base app is for projects using Electron 1.x.x, the most
common version at the time of writing. Electron 2.x.x applications should use
``org.electronjs.Electron2.BaseApp`` instead.

Command
-------

The ``command`` property indicates that a script called ``run.sh`` is to be
executed to run the application. This will be explained in further detail
later.

.. code-block:: yaml

  command: run.sh

Sandbox permissions
-------------------

The standard guidelines on sandbox permissions apply to Electron
applications. However, Electron does not currently support Wayland, so for
display access, only X11 should be used. The sample app also configures
pulseaudio for sound and enables network access:

.. code-block:: yaml

  finish-args:
    - --share=ipc
    - --socket=x11
    - --socket=pulseaudio
    - --share=network

Build options
-------------

These build options aren't strictly necessary, but can be useful if something
goes wrong.
``env`` allows setting an array of environment variables, in this case we set
``NPM_CONFIG_LOGLEVEL`` to ``info`` so that ``npm`` gives us more detailed
error messages.

.. code-block:: yaml

  build-options:
    cflags: -O2 -g
    cxxflags: -O2 -g
    env:
      NPM_CONFIG_LOGLEVEL: info

Building Node.js
----------------

The next part of the manifest is the modules list. The Electron base app
does not include Node.js, so it is necessary to build Node.js as a module.
This tutorial builds Node.js 8.11.1, as this version works with most projects
at the time of writing, but make sure to use whichever version is best for
your project.

.. code-block:: yaml

  - name: nodejs
    cleanup:
      - /include
      - /share
      - /app/lib/node_modules/npm/changelogs
      - /app/lib/node_modules/npm/doc
      - /app/lib/node_modules/npm/html
      - /app/lib/node_modules/npm/man
      - /app/lib/node_modules/npm/scripts
    sources:
      - type: archive
        url: https://nodejs.org/dist/v8.11.2/node-v8.11.2.tar.xz
        sha256: 539946c0381809576bed07424a35fc1740d52f4bd56305d6278d9e76c88f4979

Here, the cleanup step isn't strictly necessary. However, removing
documentation helps to reduce final disk size of the bundle.

The application module
----------------------

The final section of the manifest defines how the application module should
be built. This is where some of the additional logic for Electron and Node.js
can be found.

.. code-block:: yaml

  - name: electron-sample-app
    build-options:
      env:
        # Need this for electron-download to find the cached electron binary
        electron_config_cache: /run/build/electron-sample-app/npm-cache

By default, ``flatpak-builder`` doesn't allow build tools to access the
network. This means that tools which rely on downloading sources will not
work. Therefore, Node.js packages must be downloaded prior to running the
build. Setting the  ``electron_config_cache`` environment variable means
that these will be found when it comes to the build.

The next part of the manifest describes how the application should be
built. The simple buildsystem option is used, which allows a sequence of
commands to be specified, which are used for the build. The download location
and hash of the application are also specified.

.. code-block:: yaml

  buildsystem: simple
  sources:
    - type: archive
      url: https://github.com/flathub/electron-sample-app/archive/1.0.1.tar.gz
      sha256: a2feb3f1cf002a2e4e8900f718cc5c54db4ad174e48bfcfbddcd588c7b716d5b
      dest: main

Bundling NPM packages
---------------------

The next line is how NPM modules get bundled as part of Flatpaks:

.. code-block:: yaml

  - generated-sources.json

Since even simple Node.js applications depend on dozens of packages, it would
be impractical to specify all of them as part of a manifest file. A `Python
script <https://github.com/flatpak/flatpak-builder-tools/tree/master/npm>`__
has therefore been developed to download Node.js packages with NPM and
include them in an application's sources.

The Python NPM script requires a ``package-lock.json`` file. This contains
information about the packages that an application depends on, and can be
generated by running ``npm install --package-lock-only`` from an application's
root directory (the sample example contains a ``package-lock.json``, for
reference). The script is then run as follows::

  $ python3 flatpak-npm-generator.py package-lock.json

This generates the manifest JSON needed to build the NPM
packages for the application, which are outputted to a file called
``generated-sources.json``. The content of this file can be copied to
the application's manifest but, because it is often very long, it is
often best to link to it from the main manifest, which is done by adding
``generated-source.json`` as a line in the manifest section, as seen above.

Launching the app
-----------------

The Electron app is run through a simple script. This can be given any name
but must be specified in the manifest's ``"command":`` property. See below
a sample wrapper for launching app:

.. code-block:: yaml

  - type: script
    dest-filename: run.sh
    commands:
      - npm start --prefix=/app/main

Build commands
--------------

Last but not least, since the simple build option is being used, a list of
build commands must be provided. As can be seen, ``npm`` is run with the
``--offline`` option, installing dependencies from packages that have already
been cached. These are copied to ``/app/main/``. Finally the ``run.sh`` script
is installed to ``/app/bin/`` so that it will be on ``$PATH``:

.. code-block:: yaml

    build-commands:
      # Install npm dependencies
      - npm install --prefix=main --offline --cache=/run/build/electron-sample-app/npm-cache/
      # Bundle app and dependencies
      - mkdir -p /app/main /app/bin
      - cp -ra main/* /app/main/
      # Install app wrapper
      - install run.sh /app/bin/

