Electron
========

Due to the nature of `Electron <https://www.electronjs.org/>`_, building
Electron applications as Flatpaks requires a few extra steps compared with other
applications. Thankfully, several tools and resources are available which make
this much easier.

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
and navigate to the ``/flatpak`` directory in the terminal. Then
to build::

  $ flatpak-builder build org.flathub.electron-sample-app.yml --install-deps-from=flathub --force-clean --user --install

Finally, the application can be run with::

  $ flatpak run org.flathub.electron-sample-app

Basic configuration
-------------------

The first part of the sample application's manifest specifies the application's
ID. It also configures the runtime and SDK:

.. code-block:: yaml

  id: org.flathub.electron-sample-app
  runtime: org.freedesktop.Platform
  runtime-version: '24.08'
  sdk: org.freedesktop.Sdk

The Freedesktop runtime is generally the best runtime to use with Electron
applications, since it is the most minimal runtime, and other dependencies
will be specific to Electron itself.

The Electron BaseApp
---------------------

Next, the manifest specifies that the Electron BaseApp should be used, by
specifying the ``base`` and ``base-version`` properties in the application
manifest:

.. code-block:: yaml

  base: org.electronjs.Electron2.BaseApp
  base-version: '24.08'

BaseApps are described in :doc:`dependencies`. Using the Electron base
app is much faster and more convenient than manually building Electron
dependencies. It also has the advantage of reducing the amount of duplication
on users' machines, since it means that Electron is only saved once on disk.

The Node.js SDK extension
-------------------------

In order to build Electron-based apps, you need Node.js available at build time.
Flathub provides Node.js LTS versions as extensions for the SDK, so you can
install one of them and add it in your apps' manifest:

.. code-block:: yaml

  sdk-extensions:
    - org.freedesktop.Sdk.Extension.node18

Enable the extension by adding it to ``PATH``:

.. code-block:: yaml

  build-options:
    append-path: /usr/lib/sdk/node18/bin

Note that the extension name (last portion of reverse-dns notation, ``node18``
in this example) must be the same in ``sdk-extensions`` and ``append-path``.

Command
-------

The ``command`` property indicates that a script called ``run.sh`` is to be
executed to run the application. This will be explained in further detail
later.

.. code-block:: yaml

  command: run.sh

Sandbox permissions
-------------------

The standard sandbox :ref:`sandbox-permissions:Permissions guidelines` also
apply to Electron applications. However, Electron's Wayland support is still
experimental. So for display access, only X11 should be used as the default
configuration. This will make Electron use Xwayland in a Wayland session and
nothing else is required.

The sample app also configures PulseAudio for sound and enables network
access:

.. code-block:: yaml

  finish-args:
    - --share=ipc
    - --device=dri
    - --socket=x11
    - --socket=pulseaudio
    - --share=network
    - --env=ELECTRON_TRASH=gio

To allow experimental `native Wayland` support in Electron>=20, the
``--ozone-platform-hint=auto`` flag can be passed to the program. `auto`
will choose Wayland when the current session is running under Wayland and
Xwayland or X11 otherwise.

It's recommended to leave actually `enabling` Wayland up to the user for now,
i.e. set ``--socket=x11`` in the manifest. Wayland can then be tested with::

  flatpak run --socket=wayland org.flathub.electron-sample-app

Enable native Wayland support by default
````````````````````````````````````````

.. note::

  Native Wayland support in Electron is still experimental and often unstable.
  It is advised to stick with the X11/Xwayland configuration above as the
  default.

To make native Wayland the `default` for users, ``--socket=fallback-x11``
and ``--socket=wayland`` must be used in the manifest.

For Electron versions between 17 and 27, client-side window decorations under
native Wayland can be enabled by passing
``--enable-features=WaylandWindowDecorations`` to the program. For newer
versions of Electron , this isn't necessary anymore.

Electron uses ``libnotify`` on Linux to provide desktop notifications.
`Since version 0.8.0 <https://gitlab.gnome.org/GNOME/libnotify/-/merge_requests/27>`_ libnotify
automatically uses the `notification portal <https://flatpak.github.io/xdg-desktop-portal/docs/doc-org.freedesktop.portal.Notification.html>`_
when inside a sandboxed environment and ``--talk-name=org.freedesktop.Notifications``
is not required anymore. ``org.electronjs.Electron2.BaseApp`` includes
``libnotify>=0.8.0`` since ``branch/23.08``.

To ensure proper mouse cursor scaling on HiDPI displays under Wayland, the
``XCURSOR_PATH`` environment variable must be set to the host's corresponding
directories:

.. code-block:: yaml

  finish-args:
    - --env=XCURSOR_PATH=/run/host/user-share/icons:/run/host/share/icons

.. _use-correct-desktop-filename:

Using correct desktop file name
-------------------------------

It's important for Linux applications to set the correct desktop file name. If not, it can lead to problems like missing the window icon under Wayland.
By default Electron uses ``{appname}.desktop`` as desktop file name. In Flatpak the name of the desktop file must be the id of the Flatpak.
To tell Electron to use another name you need to set the ``desktopName`` key in your ``package.json`` e.g. ``"desktopName": "com.example.MyApp.desktop"``.

In case you repack a binary, you can use the ``patch-desktop-filename`` script provided by the BaseApp. Each Electron binary ships with ``resources/app.asar`` file.
You need to call ``patch-desktop-filename`` with this file as argument.
If your application is installed under ``${FLATPAK_DEST}/my-app`` you need to run ``patch-desktop-filename ${FLATPAK_DEST}/my-app/resources/app.asar``.

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


The application module
----------------------

The final section of the manifest defines how the application module should
be built. This is where some of the additional logic for Electron and Node.js
can be found.

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

  name: electron-sample-app
  buildsystem: simple
  build-options:
    env:
      XDG_CACHE_HOME: /run/build/electron-sample-app/flatpak-node/cache
      npm_config_cache: /run/build/electron-sample-app/flatpak-node/npm-cache
      npm_config_nodedir: /usr/lib/sdk/node18
      npm_config_offline: 'true'
  subdir: main
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
script <https://github.com/flatpak/flatpak-builder-tools/tree/master/node>`__
has therefore been developed to download Node.js packages with NPM or Yarn and
include them in an application's sources.

The Python script requires a ``package-lock.json`` (or ``yarn.lock``) file. This
file contains information about the packages that an application depends on, and
can be generated by running ``npm install --package-lock-only`` from an
application's root directory. The script is then run as follows::

  $ flatpak-node-generator npm package-lock.json

This generates the manifest JSON needed to build the NPM/Yarn
packages for the application, which are outputted to a file called
``generated-sources.json``. The content of this file can be copied to
the application's manifest but, because it is often very long, it is
often best to link to it from the main manifest, which is done by adding
``generated-source.json`` as a line in the manifest section, as seen above.

Launching the app
-----------------

The Electron app is run through a simple shell script that wraps
`zypak <https://github.com/refi64/zypak>`_. This script can be given any name
but must be specified in the manifest's ``command`` property. See below for a
sample wrapper to launch the app:

.. code-block:: yaml

  - type: script
    dest-filename: run.sh
    commands:
      - zypak-wrapper /app/main/electron-sample-app "$@"

Build commands
--------------

Last but not least, since the ``simple`` buildsystem is being used, a list of
build commands must be provided. As can be seen, ``npm`` is run with the
``npm_config_offline=true`` environment variable, installing dependencies from
packages that have already been cached. These are copied to ``/app/main/``.
Finally the ``run.sh`` script is installed to ``/app/bin/`` so that it will be
on ``$PATH``:

.. code-block:: yaml

    build-commands:
      # Install npm dependencies
      - npm install --offline
      # Build the app; in this example the `dist` script
      # in package.json runs electron-builder
      - |
        . ../flatpak-node/electron-builder-arch-args.sh
        npm run dist -- $ELECTRON_BUILDER_ARCH_ARGS  --linux --dir
      # Bundle app and dependencies
      - cp -a dist/linux*unpacked /app/main
      # Install app wrapper
      - install -Dm755 -t /app/bin/ ../run.sh

Note that if the application you are trying to package contains a ``build`` block in ``package.json`` with instructions for Linux, this can cause ``electron-builder`` to try to fetch additional binaries at build-time (Even if `--dir` option is used). The following example shows a configuration that will try to download AppImage binaries:

.. code-block:: json

  "build": {
    "linux": {
      "target": "AppImage",
    }
  }

The preferred way of fixing this, is not a patch, but a build-time edit using ``jq``. The following command will replace ``"target": "AppImage"`` with ``"target": "dir"``:

.. code-block:: bash

  jq '.build.linux.target="dir"' <<<$(<package.json) > package.json

Make setProgressBar and setBadgeCount work
-------------------------------------------
The `setProgressBar <https://www.electronjs.org/docs/latest/api/browser-window#winsetprogressbarprogress-options>`_ and `setBadgeCount <https://www.electronjs.org/docs/latest/api/app#appsetbadgecountcount-linux-macos>`_ functions allow showing a progress bar and a badge count in the window icon. It is implemented under Linux using the `UnityLauncherAPI <https://wiki.ubuntu.com/Unity/LauncherAPI>`_. This API is not implemented on every desktop environment. A known desktop environment which implements this is KDE.
It is also implemented by the popular `Dash to Dock <https://micheleg.github.io/dash-to-dock>`_ GNOME extension and `Plank <https://launchpad.net/plank>`_.

To make it work in Flatpak, the app needs to :ref:`use the correct desktop filename <use-correct-desktop-filename>` in its embedded `package.json` file. The Flatpak also needs the ``--talk-name=com.canonical.Unity`` permission. Electron versions earlier than v32 checks `checks if it's running on Unity or KDE <https://github.com/electron/electron/blob/fb88375ab4d2161dbf7e958a2a94c7c6d97dc84c/shell/browser/linux/unity_service.cc#L64>`_ before using the UnityLauncherAPI.
