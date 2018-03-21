Manifests
=========

The input to ``flatpak-builder`` is a JSON file that describes the parameters for building an application, as well as instructions for each of the modules that are to be built. This file is called the manifest.

The following example is the manifest for the GNOME Dictionary application. It is short, because only one module is built - the application itself::

  {
    "app-id": "org.gnome.Dictionary",
    "runtime": "org.gnome.Platform",
    "runtime-version": "3.26",
    "sdk": "org.gnome.Sdk",
    "command": "gnome-dictionary",
    "finish-args": [
       "--socket=x11",
       "--share=network"
    ],
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
  }

As can be seen, this manifest includes basic information about the application before specifying a single .tar file to be downloaded and built. More complex manifests include a sequence of modules. Module sources can be of several types, including ``.tar`` or ``.zip`` archives, Git or Bzr repositories, patch files or shell commands that are run.

Each of the properties that can be specified in a manifest file are listed in the :doc:`flatpak-builder-command-reference`, as well as the ``flatpak-builder`` man page.

.. TODO

  Add a section on the top level options, possibly called "Application metadata". Ideally this should recommend a basic set of options that developers should start with.

  Add a section on specifying the list of modules. There has to be something to say here...

Finishing
---------

Flatpaks have extremely limited access to the host environment by default. However, most applications require access to resources outside of their sandbox in order to be useful. This can be achieved with the ``finish-args`` manifest section, which allows sandbox permissions to be configured.

While there are no restrictions on which sandbox permissions an application can use, as good practice, it is recommended to use the minimum number of as permissions possible. Certain permissions, such as blanket access to the system bus (using the ``--socket=system-bus`` option) are strongly discouraged.

A list of ``finish-args`` options can be found in :doc:`sandbox-permissions`.

Cleanup
-------

After building has taken place, ``flatpak-builder`` performs a cleanup phase. This can be used to remove headers and development documentation, among other things. Two properties in the manifest file are used for this. First, a list of filename patterns can be included::

  "cleanup": [ "/include", "/bin/foo-*", "*.a" ]

The second cleanup property is a list of commands that are run during the cleanup phase::

  "cleanup-commands": [ "sed s/foo/bar/ /bin/app.sh" ]

Cleanup properties can be set on a per-module basis, in which case only filenames that were created by that particular module will be matched.

File renaming
-------------

Files that are exported by a flatpak must be prefixed using the application ID. If an application's source files are not named using this convention, flatpak-builder allows them to be renamed as part of the build process. To rename application icons, desktop files and AppData files, use the ``rename-icon``, ``rename-desktop-file`` and ``rename-appdata-file`` properties.

Example manifests
-----------------

A `complete manifest for GNOME Dictionary built from Git <https://github.com/flathub/org.gnome.Dictionary/blob/master/org.gnome.Dictionary.json>`_. It is also possible to browse `all the manifests hosted by Flathub <https://github.com/flathub>`_.
