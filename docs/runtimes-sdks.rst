Runtimes and SDKs
=================

Each Flatpak app is required to have a runtime. This provides the environment in which the app runs and includes libraries that it can access. The app's runtime must be present on a system for it to run.

Each runtime is paired with an SDK (Software Develpment Kit). For example, for the Freedesktop 1.6 runtime, there is also a Freedesktop 1.6 SDK. The SDK includes the same things as the regular runtime, but also includes all the additional development resources and tools that are required to build an app, such as build and packaging tools, header files, compilers and debuggers.

An app is built using the SDK which corresponds to the runtime that it requires when it is run. For example, an application that uses the Freedesktop 1.6 runtime in order to run must be built with the Freedesktop 1.6 SDK.

Installing an SDK
-----------------

When you come to build a Flatpak app, it is necessary to decide which runtime it will use, and then ensure that you have installed both that runtime and the matching SDK. SDKs are installed in exactly the same way as another other runtime.

Many of the examples in the rest of the Flatpak documentation use the GNOME 3.26 runtime and SDK, both of which can be obtained from Flathub. To install them, run::

  $ flatpak install flathub org.gnome.Platform//3.26 org.gnome.Sdk//3.26
