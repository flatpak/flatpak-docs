Dependencies
============

Flatpak provides a number of different options for how applications can depend
on other software. When setting out to build an application with Flatpak
for the first time, it is therefore necessary to decide how application
dependencies will be organized.

This page outlines what the options are, and provides guidance on when to
use each one.

Runtimes
--------

As was described in :doc:`basic-concepts`, runtimes provide basic
dependencies that can be used by applications. They also provide the
environment that applications run in. Flatpak requires each application to
specify a runtime. Therefore, one of the first decisions you need to make
when building an application with Flatpak, is which runtime it will use.

An overview of the runtimes that are available can be found in the
:doc:`available-runtimes` page. There are deliberately only a small number
of runtimes to choose from. Typically, runtimes are picked on the basis of
which dependencies an application requires. If a runtime exists that provides
libraries that you plan on using, this is usually the correct runtime to use!

.. tip::

  Runtimes require regular maintenance, and application developers should
  generally not consider creating their own.

Runtimes are automatically installed for users when they install an
application, and build tools can also automatically install them for
you (``flatpak-builder``'s ``--install-deps-from`` option is useful for
this). However, if you do need to manually install your chosen runtime,
this can be done in the same way as installing an application, with the
``flatpak install`` command. For example, the command to install the GNOME
43 runtime is::

  $ flatpak install flathub org.gnome.Platform//43

Bundling
--------

One of the key advantages of Flatpak is that it allows application authors
to bundle whatever libraries or dependencies that they want. This means
that developers aren't constrained by which libraries are available through
Linux distributions.

When it comes to building an application for the first time, you will need
to decide which dependencies to bundle. This can include:

- libraries that aren't in your chosen runtime
- different versions of libraries that are in your chosen runtime
- patched versions of libraries
- data or other resources that form part of the application

As will be seen, bundled dependencies can be automatically downloaded as
part of the build process. It is also possible to apply patches and perform
other transformations.

While bundling is very powerful and flexible, it also places a greater
maintenance burden on the application developer. Therefore, while it is
possible to bundle as much as you would like, it is generally recommended to
try and keep the number of bundled modules as low as possible. If a dependency
is available as part of a runtime, it is generally better to use that version
rather than bundle it yourself.

The specifics of how to bundle libraries is covered in the :doc:`manifests`
section.

BaseApps
---------

Runtimes and bundling are the two main ways in which dependencies are handled
with Flatpak. They allow applications to rely on stable collections of
dependencies on the one hand, and to have flexibility and control on the other.

However, in some cases, dependencies come as part of a bigger framework or
toolkit, which doesn't fit into a runtime but which is also cumbersome to
manually bundle as a series of individual modules. This is where *BaseApps*
come in.

BaseApps contain collections of bundled dependencies which can then be
bundled as part of an application. They don't get rebuilt as part of the
build process, which makes building faster (particularly when bundling large
dependencies). And because each BaseApp is only built once, it is guaranteed
to be identical wherever it is used, so it will only be saved once on disk.

BaseApps are a relatively specialized concept and only some applications
need to use them (the most common BaseApp is used for `Electron applications
<https://github.com/flathub/io.atom.electron.BaseApp>`_). However, if your
application uses a large, complex or specialized framework, it is a good
idea to check for available BaseApps before you start building.

Extensions
----------

Runtimes and applications can define extension points which allow optional
additional runtimes to be mounted at a specified location inside the sandbox
when they are present on the system. Typical uses for extensions include
translations for applications, debuginfo for SDKs, or adding more functionality
to the application. Some software refers to these extensions as "Add-ons".

For an extension to be loaded in the application or runtime, an extension
point first needs to be defined in the application or runtime in 
question.

By convention, extension points follow the ID of the application or 
runtime in question, followed by a generic term for the extension. 
For example, the OBS Studio flatpak may define an extension point 
``com.obsproject.Studio.Plugin``, where "Plugin" is the generic term 
prefixed by the application ID.

To see available extension points, it's best to look at the application
or runtime manifest.

Any extension now choosing to be loaded inside the OBS Studio flatpak 
must prefix their ID by ``com.obsproject.Studio.Plugin`` for example
``com.obsproject.Studio.Plugin.Gstreamer``.

Some extension names having special significance are discussed below. 

- ``.Debug`` is used for Debug extensions by applications, runtimes and 
  SDKs. They are used for debugging purposes. Every application or runtime 
  built with ``flatpak-builder`` produces a ``.Debug`` extension unless 
  specifically disabled in the manifest.

  A ``.Debug`` extension will be needed when generating useful 
  backtraces. This is explained more in :doc:`debugging`.

- ``.Locale`` is used for Locale extensions by applications or runtimes.
  They add support for more languages to the parent application or runtime.
  These are usually partially downloaded by ``flatpak`` based on the 
  configured system locale. Every application or runtime built with 
  ``flatpak-builder`` produces a ``.Locale`` extension unless specifically 
  disabled in the manifest.

- ``.Sources`` is used for Sources extension by application or runtime.
  They are used to bundle sources of the modules used in the application 
  or runtime in question. ``flatpak-builder`` will produce a ``.Sources`` 
  extension prefixed by ID when ``--bundle-sources`` is used.

Please visit :doc:`extension` for a guide on how to create 
extension points and extensions.
