Flatpak Builder
===============

Most applications require additional dependencies that aren't provided by their runtimes. Flatpak allows these dependencies to be bundled as part of the application itself. In order to do this, each dependency must be built inside the application build directory. The ``flatpak-builder`` tool automates this multi-step build process, making it possible to build all application modules with a single command. 

flatpak-builder expects modules to be built in the standard manner by following what is called the `Build API <https://github.com/cgwalters/build-api/>`_. This requires modifying modules to follow the build API, if they don't already.

Manifests
---------

The input to flatpak-builder is a JSON file that describes the parameters for building an application, as well as each of the modules to be bundled. This file is called the manifest. Module sources can be of several types, including ``.tar`` or ``.zip`` archives, Git or Bzr repositories, patch files or shell commands that are run.

The GNOME Dictionary manifest is short, because the only module is the application itself::

  {
    "app-id": "org.gnome.Dictionary",
    "runtime": "org.gnome.Platform",
    "runtime-version": "3.22",
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
            "url": "https://download.gnome.org/sources/gnome-dictionary/3.20/gnome-dictionary-3.20.0.tar.xz",
            "sha256": "efb36377d46eff9291d3b8fec37baab2355f9dc8bc7edb791b6a625574716121"
          }
        ]
      }
    ]
  }

As can be seen, this manifest includes basic information about the application before specifying a single .tar file to be downloaded and built. More complex manifests include a sequence of modules.

Cleanup
-------

After building has taken place, flatpak-builder performs a cleanup phase. This can be used to remove headers and development documentation, among other things. Two properties in the manifest file are used for this. First, a list of filename patterns can be included::

  "cleanup": [ "/include", "/bin/foo-*", "*.a" ]

The second cleanup property is a list of commands that are run during the cleanup phase::

  "cleanup-commands": [ "sed s/foo/bar/ /bin/app.sh" ]

Cleanup properties can be set on a per-module basis, in which case only filenames that were created by that particular module will be matched.

File renaming
-------------

Files that are exported by a flatpak must be prefixed using the application ID. If an application's source files are not named using this convention, flatpak-builder allows them to be renamed as part of the build process. To rename application icons, desktop files and AppData files, use the ``rename-icon``, ``rename-desktop-file`` and ``rename-appdata`` properties.

Splitting things up
-------------------

By default, flatpak-builder splits off translations and dubug information into separate .Locale and .Debug extensions. These 'standard' extension points are then added to the application's metadata file. You can turn this off with the ``separate-locales`` and ``no-debuginfo`` keys, but there shouldn't be any reason for it.

When flatpak-builder exports the build into a repository, it automatically includes the .Locale and .Debug extensions. If you do the exporting manually, don't forget to include them.

Example
-------

You can try flatpak-builder for yourself, using the repository that was created in the `previous section <building-simple-apps.html>`_. To do this, place the manifest JSON from above into a file called ``org.gnome.Dictionary.json`` and run the following command::

  $ flatpak-builder --repo=repo dictionary2 org.gnome.Dictionary.json

This will:

* Create a new directory (called dictionary2)
* Download and verify the Dictionary source code
* Build and install the source code, using the SDK rather than the host system
* Finish the build, by setting permissions (in this case giving access to X and the network)
* Export the resulting build to the tutorial repository, which contains the Dictionary app that was previously installed

flatpak-builder will also do some other useful things, like creating a separately installable debug runtime (called `org.gnome.Dictionary.Debug` in this case) and a separately installable translation runtime (called ``org.gnome.Dictionary.Locale``).

If you completed the tutorial in `Building Simple Apps <building-simple-apps.html>`_, you can update the Dictionary application you installed with the new version that was built and exported by flatpak-builder::

  $ flatpak --user update org.gnome.Dictionary

To check that the application has been successfully updated, you can compare the sha256 commit of the installed app with the commit ID that was printed by flatpak-builder::

  $ flatpak info org.gnome.Dictionary
  $ flatpak info org.gnome.Dictionary.Locale

And finally, you can run the new version of the Dictionary app::

  $ flatpak run org.gnome.Dictionary

Example manifests
-----------------

A `complete manifest for GNOME Dictionary built from Git <https://git.gnome.org/browse/gnome-apps-nightly/tree/org.gnome.Dictionary.json>`_ is available, in addition to `manifests for a range of other GNOME applications <https://git.gnome.org/browse/gnome-apps-nightly/tree/>`_.
