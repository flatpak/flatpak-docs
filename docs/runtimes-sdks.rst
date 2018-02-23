Runtimes and SDKs
=================

Each Flatpak app is required to have a runtime. This provides the environment that the app runs in, and includes a set of libraries that it can access. The app's runtime must be present on a system for it to run.

Each runtime is paired with an SDK (Software Develpment Kit). For example, for the Freedesktop 1.6 runtime, there is also a Freedesktop 1.6 SDK. The SDK includes the same things as the regular runtime, but also includes all the additional development resources and tools that are required to build an app, such as build and packaging tools, header files, compilers and debuggers.

Applications must be built with the SDK that corresponds to their runtime. For example, an application that uses the Freedesktop 1.6 runtime in order to run must be built with the Freedesktop 1.6 SDK.

Installing an SDK
-----------------

When you come to build a Flatpak app, it is necessary to decide which runtime it will use, and then ensure that you have installed both that runtime and the matching SDK.

SDKs are installed in the same way as any other runtime. For example, the command to install the GNOME 3.25 runtime and SDK is::

  $ flatpak install flathub org.gnome.Platform//3.26 org.gnome.Sdk//3.26

A number of the examples in these docs use this runtime and SDK, so it is a good idea to try this command yourself.
