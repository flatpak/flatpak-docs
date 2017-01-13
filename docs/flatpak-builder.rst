Flatpak Builder
===============

If an application requires additional dependencies that aren't provided by its runtime, Flatpak allows them to be bundled as part of the app itself. This requires building each module inside the application build directory, which can be a lot of work. The ``flatpak-builder`` tool can automate this multi-step build process.

flatpak-builder takes care of the routine commands used to build an app and any bundled libraries, thus allowing application building to be automated. To do this, it expects modules to be built in a standard manner by following what is called the `Build API <https://github.com/cgwalters/build-api/>`_. If any modules don't conform to this API, they will need to be modified.

Manifests
---------

The input to flatpak-builder is a json file that describes the parameters for building an app, as well as each of the modules to be bundled. This file is called the manifest. Module sources can be of several types, including ``.tar`` or ``.zip`` archives, Git or Bzr repositories, patch files or shell commands that are run.

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
            "url": "https://download.gnome.org/sources/gnome-dictionary/3.22/gnome-dictionary-3.22.0.tar.xz",
            "sha256": "efb36377d46eff9291d3b8fec37baab2355f9dc8bc7edb791b6a625574716121"
          }
        ]
      }
    ]
  }

Cleanup
-------

flatpak-builder performs a cleanup phase after the build, which can be used to remove headers and development docs, among other things. Two properties in the manifest file can be used for this. First, a list of filename patterns can be included::

  "cleanup": [ "/include", "/bin/foo-*", "*.a" ]

The second cleanup property is a list of commands that are run during the cleanup phase::

  "cleanup-commands": [ "sed s/foo/bar/ /bin/app.sh" ]

Cleanup properties can be set on a per-module basis, and will then only match filenames that were created by that particular module.

File renaming
-------------

Files that are exported by a flatpak must be named using the application ID. However, application's source files will typically not follow this convention. To get around this, flatpak-builder allows renaming application icons, desktop files and AppData files as a part of the build process, using the ``rename-icon``, ``rename-desktop-file`` and ``rename-appdata`` properties.

Splitting things up
-------------------

By default, flatpak-builder splits off translations into a separate .Locale runtime, and debuginfo into a .Debug runtime, and adds these 'standard' extension points to the application metadata. You can turn this off with the ``separate-locales`` and ``no-debuginfo`` keys, but there shouldn't be any reason for it.

When flatpak-builder exports the build into a repository, it automatically includes the .Locale`` and .Debug runtimes. If you do the exporting manually, don't forget to include them.

Example
-------

You can try flatpak-builder for yourself, using the repository that was created in the previous section. To do this, place the manifest json from above into a file called ``org.gnome.Dictionary.json`` and run the following command::

  $ flatpak-builder --repo=repo dictionary2 org.gnome.Dictionary.json
  
This will:

* Create a new directory (called dictionary2)
* Download and verify the Dictionary source code
* Build and install the source code, using the SDK rather than the host system
* Finish the build, by setting permissions (in this case giving access to X and the network)
* Export the resulting build to the tutorial repository, which contains the Dictionary app that was previously installed

flatpak-builder will also do some other useful things, like creating a separately installable debug runtime (called `org.gnome.Dictionary.Debug` in this case) and a separately installable translation runtime (called ``org.gnome.Dictionary.Locale``).

It is now possible to update the installed version of the Dictionary application with the new version that was built and exported by flatpak-builder::

  $ flatpak --user update org.gnome.Dictionary

To check that the application has been successfully updated, you can compare the sha256 commit of the installed app with the commit ID that was printed by flatpak-builder::

  $ flatpak info org.gnome.Dictionary
  $ flatpak info org.gnome.Dictionary.Locale
  
And finally, you can run the new version of the Dictionary app::

  $ flatpak run org.gnome.Dictionary
  
Example manifests
-----------------

A `complete manifest for GNOME Dictionary built from Git <https://git.gnome.org/browse/gnome-apps-nightly/tree/org.gnome.Dictionary.json>`_ is available, in addition to `manifests for a range of other GNOME applications <https://git.gnome.org/browse/gnome-apps-nightly/tree/>`_.
