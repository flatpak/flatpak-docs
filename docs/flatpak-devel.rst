Flatpak as a developer platform
===============================

This sections contains an overview on how Flatpak can be used as part
of a developer workflow.

CI Integration
--------------

Flatpak can be integrated with the CI pipelines and bundles can be
produced as artifacts for quick testing or it can be exported to a
Flatpak repository on a webserver to provide a Nightly Flatpak package.

The CI setup will vary based on the code host and the CI service being
used. A basic workflow for exporting to Gitlab pages is provided in
:doc:`hosting-a-repository` which can be used as a starting point.

`GNOME <https://gitlab.gnome.org/GNOME/Initiatives/-/wikis/DevOps-with-Flatpak#basic-ci>`_
and `KDE <https://develop.kde.org/docs/packaging/flatpak/publishing/#publishing-to-kdes-nightly-repositories>`_
also has their own respective CI setup for doing this, which can be
also be used for inspiration.

`Flatpak Github Actions <https://github.com/flathub-infra/flatpak-github-actions>`_
can be used for GitHub.

Running tests
-------------

Flatpak Builder can also run tests with ``run-tests, test-rule, test-commands, test-args``.

``run-tests: true`` in a module will run tests for that module after
installing.

``test-rule`` is the default target to build when running tests. For
``meson`` an ``cmake-ninja`` buildsystems it defaults to ``test``,
otherwise ``check`` for ``cmake`` and ``autotools``. Setting it to an
empty string like ``test-rule: ''`` will disable the target.

``test-commands`` is an array of additional commands that will be run
during tests.

.. code-block:: yaml

  run-tests: true
  test-rule: ''
  test-commands:
     - xvfb-run tests/test_foo

``test-args`` is used to provide finish-args for tests. These do not
affect the normal installation.

.. code-block:: yaml

  build-options:
      test-args:
          - "--socket=x11"
          - "--share=network"

Parallel nightly and stable applications
----------------------------------------

In general stable and nightly versions or any two parallel branches
of an application should have different application IDs. This prevents many
potential conflicts such as incompatible configuration files or
overlapping well-known D-Bus names. This is mandatory if the application
uses its own well-known D-Bus name.

A standard way is to use the ``.Devel`` suffix to the original Flatpak ID.
The internal application ID, appstream ID, launchable, icon tags,
icon files, desktop files, Dbus service names etc. must follow the new
Flatpak ID for everything to work seamlessly for the user.

This can be integrated with buildsystem `profiles`, so simply building
with ``-Dprofile=Devel`` for example will build the Nightly application
and the default will build the stable appliction.

Buildsystem handling
^^^^^^^^^^^^^^^^^^^^

This is an example for the Meson buildsystem.

The ``src`` subdirectory will contain the main source code of the
application and ``data`` subdirectory will have application metadata
such as desktop file, metainfo file, icons and the ``po`` subdirectory
will have translation files.

The root ``meson.build`` parts should be something like this:

.. code-block:: meson

  i18n = import('i18n')

  profile = get_option('profile')
  if profile == 'development'
    application_id = 'org.example.coolapp.Devel'
  else
    application_id = 'org.example.coolapp'
  endif

  config_h = configuration_data()
  config_h.set_quoted('APPLICATION_ID', application_id)
  config_h.set_quoted('PROFILE', profile)
  config_h.set_quoted('GETTEXT_PACKAGE', 'coolapp')
  config_h.set_quoted('PACKAGE_VERSION', meson.project_version())
  configure_file(
     output: 'config.h',
     configuration: config_h,
  )

  configuration_inc = include_directories('.')

  subdir('data')
  subdir('src')
  subdir('po')

  summary({
    'Profile': get_option('profile'),
  }, section: 'Development')


The root ``meson-options.txt`` should have something like::

  option('profile', type: 'combo', choices: ['default', 'development'], value: 'default')

Now under ``src/meson.build`` or in any subdirectory this can be
specified in a target.

.. code-block:: meson

  coolapp = executable('coolapp', 'main.c',
      include_directories: configuration_inc,
      dependencies: libcoolapp_dep,
      install: true,
  )

Now in ``data/meson.build``, the icon, desktop file and metainfo file
handling can be specified.

Place a desktop template file called ``org.example.coolapp.desktop.in.in``
in ``data/`` with the contents

.. code-block:: ini

  [Desktop Entry]
  Name=Cool App
  Exec=coolapp
  Icon=@icon@
  Terminal=false
  Type=Application
  Categories=Utility;
  StartupNotify=true


The corresponding meson bits for the desktop file should be

.. code-block::

  desktop_conf = configuration_data()
  desktop_conf.set('icon', application_id)
  desktop_file = i18n.merge_file(
    input: configure_file(
      input: files('org.example.coolapp.desktop.in.in'),
      output: 'org.example.coolapp.desktop.in',
      configuration: desktop_conf,
    ),
    output: '@0@.desktop'.format(application_id),
    type: 'desktop',
    po_dir: '../po',
    install: true,
    install_dir: get_option('datadir') / 'applications',
  )
  desktop_utils = find_program('desktop-file-validate', required: false)
  if desktop_utils.found()
    test('Validate desktop file', desktop_utils, args: [desktop_file])
  endif

Appstream can be handled in the same ``meson.build``. Place a metainfo
template file in ``data/``. Any tag containing part of the application
ID should be templated. Ususally for desktop applications this involves
only the ``id`` and ``launchable`` tag, but the ``icon`` tag, if present
should also use it.

.. code-block:: xml

  <?xml version="1.0" encoding="UTF-8"?>
  <component type="desktop">
    <id>@appid@</id>
    ...
    <launchable type="desktop-id">@appid@.desktop</launchable>
    ...
  </component>

The corresponding meson.build part for metainfo should be

.. code-block::

  metainfo_conf = configuration_data()
  metainfo_conf.set('appid', application_id)
  appstream_file = i18n.merge_file(
    input: configure_file(
      input: files('org.example.coolapp.metainfo.xml.in.in'),
      output: 'org.example.coolapp.metainfo.xml.in',
      configuration: metainfo_conf
    ),
    output: '@0@.metainfo.xml'.format(application_id),
    po_dir: '../po',
    install: true,
    install_dir: get_option('datadir') / 'metainfo',
  )

  appstreamcli = find_program('appstreamcli', required: false)
  if (appstreamcli.found())
    test('Validate metainfo file',
      appstreamcli,
      args: ['validate', '--no-net', '--explain', appstream_file],
      workdir: meson.current_build_dir()
    )
  endif

Finally icons can be placed in ``data/icons/hicolor/{scalable, symbolic}/apps``.
Two icons should be provided ``org.example.coolapp.svg`` and ``org.example.coolapp.Devel.svg``.
The meson.build should have

.. code-block:: meson

  icondir = join_paths('icons', 'hicolor', 'scalable', 'apps')
  install_data(
    join_paths(icondir, '@0@.svg'.format(application_id)),
    install_dir: join_paths(datadir, icondir),
    rename: '@0@.svg'.format(application_id)
  )

  icondir = join_paths('icons', 'hicolor', 'symbolic', 'apps')
  install_data(
    join_paths(icondir, 'org.example.coolapp-symbolic.svg'),
    install_dir: join_paths(datadir, icondir),
    rename: '@0@-symbolic.svg'.format(application_id)
  )

Any gschemea file should also be similarly handled if present.

.. code-block::

  install_data('org.example.coolapp.gschema.xml',
    rename: '@0@.gschema.xml'.format(application_id),
    install_dir: get_option('datadir') / 'glib-2.0' / 'schemas',
  )

Any dbus service file if present should be handled. The ``org.example.coolapp.service.in``
should have ``Name=@appid@``

.. code-block:: meson

  service_conf = configuration_data()
  service_conf.set('appid', application_id)
  service_conf.set('bindir', join_paths(prefix, bindir))
  configure_file(
    input: 'org.example.coolapp.service.in',
    output: '@0@.service'.format(application_id),
    configuration: service_conf,
    install_dir: servicedir
  )

The same pattern can be used for search provider files if present.
The ``org.example.coolapp.service.ini.in`` should look like

.. code-block:: ini

  DesktopId=@appid@.desktop
  BusName=@appid@
  ObjectPath=/org/gnome/AppName@profile@/SearchProvider

and the corresponding meson.build:

.. code-block:: meson

  search_provider_conf = configuration_data()
  search_provider_conf.set('appid', application_id)
  search_provider_conf.set('profile', profile)
  configure_file(
    configuration: search_provider_conf,
    input: files('org.example.coolapp.service.ini.in'),
    install_dir: join_paths(datadir, 'gnome-shell', 'search-providers'),
    output: '@0@.search-provider.ini'.format(application_id)
  )

Application handling
^^^^^^^^^^^^^^^^^^^^

The correct application ID must be supplied when initiating the
application. For example in ``main.c``

.. code-block:: c

  #include "config.h"

  ...

  int
  main (int argc, char **argv)
  {

    ...

    g_set_prgname (APPLICATION_ID);
    gtk_window_set_default_icon_name (APPLICATION_ID);

    coolapp = g_object_new (APP_NAME_TYPE_APPLICATION,
                            "application-id", APPLICATION_ID,
                            "flags", G_APPLICATION_HANDLES_OPEN,
                            NULL);

    g_application_run (G_APPLICATION (coolapp), argc, argv);

    return 0;
  }

Similarly it can be handled in the about dialogue of the application and
elsewhere.

Manifest handling
^^^^^^^^^^^^^^^^^

Finally there must be two manifests for Devel and stable - ``org.example.coolapp.Devel.yaml``
and ``org.example.coolapp.yaml`` respectively.

The manifest must have the correct ``id`` property. The devel manifest
should use ``id: org.example.coolapp.Devel`` and the stable manifest
should use ``id: org.example.coolapp``.

In ``config-opts`` for the application module, ``-Dprofile=Devel`` can be
passed to build the Devel application.

In case, buildsystem handling for desktop file, icon and metainfo file
is absent ``rename-desktop-file, rename-appdata-file, rename-mime-file, rename-icon``
can be used rename the files to match the ``.Devel`` application ID.

``desktop-file-name-prefix`` or ``desktop-file-name-suffix`` can be used
to add a prefix or suffix to the desktop file name respectively.

.. code-block:: yaml

  id: org.example.coolapp.Devel
  ...
  rename-desktop-file: org.example.coolapp.desktop
  rename-appdata-file: org.example.coolapp.metainfo.xml
  rename-icon: org.example.coolapp
  desktop-file-name-suffix: ' (Nightly)'


The main drawback of modifying it in place with Flatpak Builder is that,
the application ID, D-Bus names and configuration location cannot be
changed. This might cause integration issues with desktop environment,
potential configuration conflicts, or the application might not run in
some cases when the D-Bus name is already in use. This would prevent
running the stable and nightly applications parallely.


In case the application ID remains static, for GTK based applications
the ``--name`` flag can be passed to the main binary with the correct
application ID; ``--class`` and ``StartupWMClass`` in the
desktop file to fix the window class.


Additional tools
----------------

- `Electron Builder <https://www.electron.build/flatpak.html>`_
  supports exporting single file Flatpak bundles. Please also see the
  Electron application packaging guide :doc:`electron`.

- `GNOME Builder <https://apps.gnome.org/Builder/>`_ is an IDE that can
  integrate with the development workflow of GNOME applications.
  `Qt Creator <https://flathub.org/apps/io.qt.QtCreator>`_ and `Qt Design Studio <https://flathub.org/apps/io.qt.qtdesignstudio>`_
  are both available as Flatpaks from Flathub along with a variety of
  IDEs.

- `Freedesktop SDK <https://gitlab.com/freedesktop-sdk/freedesktop-sdk/-/wikis/Freedesktop-SDK-Flatpak-repository>`_
  and `GNOME Nightly <https://nightly.gnome.org/>`_ hosts nightly
  versions of the ``org.freedesktop.Platform`` and ``org.gnome.Platform``
  runtimes and SDKs.

  `GNOME OS <https://os.gnome.org/>`_ is an immutable Flatpak first system
  that can also be used to build and test applications on upcoming
  GNOME versions.

- Freedesktop SDK also builds Mesa from the git main branch. Please see
  the `docs <https://gitlab.com/freedesktop-sdk/freedesktop-sdk/-/wikis/mesa-git>`_
  on how to use that.

- `Flathub hosts <https://github.com/flathub?q=org.freedesktop.Sdk.Extension&type=all&language=&sort=>`_
  extensions for lots of tooling and languages that can be used to build
  applications.
