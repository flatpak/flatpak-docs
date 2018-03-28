Building Concepts and Setup
===========================

The :doc:`introduction` and page on :doc:`first-build` have already shown how applications get built as Flatpaks, and have introduced many of the core ideas. This page describes some of these ideas and concepts in more detail. In doing so, it provides guidance on picking a runtime, getting setup to build applications, and when to bundle dependencies yourself.

Runtimes
--------

As was described in the :doc:`introduction`, runtimes provide basic dependencies that can be used by applications. They also provide the environment that applications run in.

Flatpak requires each application to specify a runtime, and this runtime must be present on a system for it to run. Therefore, one of the first decisions you need to make when building an application with Flatpak, is which runtime it will use.

An overview of the runtimes that are available can be found in the :doc:`available-runtimes` page. There are deliberately only a small number of runtimes to choose from. Typically, runtimes are picked on the basis of which dependencies an application requires. If a runtime exists that provides libraries that you plan on using, this is usually the correct runtime to use!

.. tip::

  Runtimes require regular maintenance, and application developers should generally not consider creating their own.

Runtimes are automatically installed for users when they install an application, and build tools can also automatically install them for you (``flatpak-builder``'s ``--install-deps-from`` option is useful for this). However, if you do need to manually install your chosen runtime, this can be done in the same way as installing an application, with the ``flatpak install`` command. For example, the command to install the GNOME 3.26 runtime is::

  $ flatpak install flathub org.gnome.Platform//3.26

Software Development Kits (SDKs)
--------------------------------

Each runtime is paired with an SDK (Software Develpment Kit). For example, the GNOME 3.26 runtime is accompanied by the GNOME 3.26 SDK. The SDK includes the same things as the regular runtime, but also includes all the development resources and tools that are required to build an application, such as build and packaging tools, header files, compilers and debuggers.

Applications must be built with the SDK that corresponds to their runtime, so that applications that use the GNOME 3.26 runtime must be built with the GNOME 3.26 SDK.

Like runtimes, SDKs will sometimes be automatically installed for you, but if you do need to manually install them, they are installed in the same was as applications and runtimes, such as::

 $ flatpak install flathub org.gnome.Sdk//3.26

Bundling
--------

One of the key advantages of Flatpak is that it allows application authors to bundle whatever libraries or dependencies that they want. This means that developers aren't constrained by which libraries are available through Linux distributions. It also provides flexibility, allowing particular versions of libraries to be used, or the use of libraries that have been patched.

While bundling is very powerful and flexible, it also places a greater maintenance burden on the application developer. Therefore, while it is possible to bundle as much as you would like, it is generally recommended to try and keep the number of bundled modules as low as possible. If a dependency is available as part of a runtime, it is generally better to use the version from the runtime rather than bundle it yourself.

The specifics of how to bundle libraries is covered in the :doc:`manifests` section.

Base apps
---------

Flatpak allows almost any module to be bundled as part of an application, even other applications. Typically, this is done with special-purpose applications that have been created in order to be bundled. These applications, called *base apps*, contain dependencies or frameworks that can be used by other applications, in order to share dependencies.

Base apps don't get rebuilt as part of the build process, which makes building faster (particularly when bundling large dependences). And because each base app is only built once, it is guaranteed to be identical wherever it is used, so it will only be saved once on disk.

A number of base apps are available and, unlike runtimes, they can be built and published as required.
