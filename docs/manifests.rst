Manifests
=========

The input to ``flatpak-builder`` is a JSON or YAML file that describes the
parameters for building an application, as well as instructions for each of the
modules that are to be built. This file is called the manifest.

This page provides information and guidance on how to use manifests, including
an explanation of the most common parameters that can be specified. It is
recommended to have followed the :doc:`first-build` tutorial before reading
this section, and to be familiar with :doc:`flatpak-builder`.

Manifest files should be named using the application ID. For example, the
manifest file for GNOME Dictionary is named ``org.gnome.Dictionary.json``. This
page uses this manifest file, which was introduced in :doc:`first-build`, for
all its examples.

A complete list of all the properties that can be specified in manifest files
can be found in the :doc:`flatpak-builder-command-reference`, as well as the
``flatpak-manifest`` man page.

Basic properties
----------------

Each manifest file should specify basic information about the application that
is to be built, including the ``app-id``, ``runtime``, ``runtime-version``,
``sdk`` and ``command`` parameters. These properties are typically specified at
the beginning of the file.

For example, the GNOME Dictionary manifest includes:

.. code-block:: json

  "app-id": "org.gnome.Dictionary",
  "runtime": "org.gnome.Platform",
  "runtime-version": "3.26",
  "sdk": "org.gnome.Sdk",
  "command": "gnome-dictionary",

Specifying a runtime and runtime version allows that the runtime that is needed
by your application to be automatically installed on users' systems.

File renaming
-------------

As was described in the :doc:`introduction`, exports are application files that
are made available to the host, and include things like the application's
``.desktop`` file and icon.

The names of files that are exported by a Flatpak must prefixed using the
application ID, such as ``org.gnome.Dictionary.desktop``. The best way to do
this is to rename these files directly in the application's source.

If renaming exported files to use the application ID is not possible,
``flatpak-builder`` allows them to be renamed as part of the build process.
This can be done by specifying one of the following properties in the manifest:

- ``rename-icon`` - rename the application icon
- ``rename-desktop-file`` - rename the ``.desktop`` filename
- ``rename-appdata-file`` - rename the AppData file

Each of these properties accepts the name of the source file to be renamed.
``flatpak-builder`` then automatically renames the file to match the
application ID. Note that this renaming method can introduce internal naming
conflicts, and that renaming files in tree is therefore the most reliable
approach.

Finishing
---------

Applications that are run with Flatpak have extremely limited access to the
host environment by default, but applications require access to resources
outside of their sandbox in order to be useful. Finishing is the build stage
where the application's sandbox permissions are specified, in order to give
access to these resources.

The finishing manifest section uses the ``finish-args`` property, which can be
seen in the Dictionary manifest file:

.. code-block:: json

    "finish-args": [
       "--socket=x11",
       "--share=network"
    ],

As was explained in :doc:`first-build`, these two finishing properties give the
application access to the X11 display server and to the network. Guidance on
which permissions to use can be found in :doc:`sandbox-permissions`, and a full
list of ``finish-args`` options can be found in
:doc:`sandbox-permissions-reference`.

Cleanup
-------

The cleanup property can be used to remove files that are produced by the build
process but which aren't wanted as part of the application, such as headers or
developer documentation. Two properties in the manifest file are used for this.
First, a list of filename patterns can be included::

  "cleanup": [ "/include", "/bin/foo-*", "*.a" ]

The second cleanup property is a list of commands that are run during the
cleanup phase::

  "cleanup-commands": [ "sed s/foo/bar/ /bin/app.sh" ]

Cleanup properties can be set on a per-module basis, in which case only
filenames that were created by that particular module will be matched.

Modules
-------

The module list specifies each of the modules that are to be built as part of
the build process. One of these modules is the application itself, and other
modules are dependencies and libraries that are bundled as part of the Flatpak.
While simple applications may only specify one or two modules, and therefore
have short modules sections, some applications can bundle numerous modules and
therefore have lengthy modules sections.

GNOME Dictionary's modules section is short, since it just contains the
application itself, and looks like:

.. code-block:: json

  "modules": [
    {
      "name": "gnome-dictionary",
      "sources": [
        {
          "type": "archive",
          "url": "https://download.gnome.org/sources/gnome-dictionary/3.26/gnome-dictionary-3.26.0.tar.xz",
          "sha256": "387ff8fbb8091448453fd26dcf0b10053601c662e59581097bc0b54ced52e9ef"
        }
      ]
    }
  ]

As can be seen, each listed module has a ``name`` (which can be freely
assigned) and a list of ``sources``. Each source has a ``type``, and available
types include:

 - ``archive`` - ``.tar`` or ``.zip`` archive files
 - ``git`` - Git repositories
 - ``bzr`` - Bazaar repositories
 - ``file`` - local file (these are copied into the source directory)
 - ``dir`` - local directory (these are copied into the source directory)
 - ``script`` - an array of shell commands (these are put in a shellscript
   file)
 - ``shell`` - an array of shell commands that are run during source extraction
 - ``patch`` - a patch (are applied to the source directory)
 - ``extra-data`` - data that can be downloaded at install time; this can
   include archive or package files

Different properties are available for each source type, which are listed in
the :doc:`flatpak-builder-command-reference`.

Supported build systems
```````````````````````

Modules can be built with a variety of build systems, including:

- `autotools <https://www.gnu.org/software/automake/manual/html_node/Autotools-Introduction.html>`_
- `cmake <https://cmake.org/>`_
- `cmake-ninja <https://cmake.org/cmake/help/v3.0/generator/Ninja.html>`_, `meson <http://mesonbuild.com/>`_
- the "`Build API <https://github.com/cgwalters/build-api/>`_"

A "simple" build method is also available, which allows a series of commands to
be specified.

Example manifests
-----------------

A `complete manifest for GNOME Dictionary built from Git
<https://github.com/flathub/org.gnome.Dictionary/blob/master/org.gnome.Dictionary.json>`_.
It is also possible to browse `all the manifests hosted by Flathub
<https://github.com/flathub>`_.
