Runtimes and SDKs
=================

Each Flatpak app is required to have a runtime. This provides the environment that the app runs in, and includes a set of libraries that it can access. The app's runtime must be present on a system for it to run.

Each runtime is paired with an SDK (Software Develpment Kit). For example, for the Freedesktop 1.6 runtime, there is also a Freedesktop 1.6 SDK. The SDK includes the same things as the regular runtime, but also includes all the additional development resources and tools that are required to build an app, such as build and packaging tools, header files, compilers and debuggers.

Applications must be built with the SDK that corresponds to their runtime. For example, an application that uses the Freedesktop 1.6 runtime in order to run must be built with the Freedesktop 1.6 SDK.

..
  TODO: describe how SDK extensions work

Choosing a runtime
------------------

When you come to build a Flatpak app, you will need to decide which runtime it will use. An overview of the runtimes that are available can be found in the :doc:`available-runtimes` page. There are deliberately only a small number of runtimes to choose from.

Runtimes require regular maintenance, and application developers should generally not consider creating their own.

..
   TODO: include details of what the alternatives to creating a runtime are.
   Might want to mention meta-apps.

Installing a runtime and SDK
----------------------------

Once you have chosen a runtime for your application, it is necessary to install it as well as the matching SDK. (Runtimes and SDKs are installed in exactly the same way.)

For example, the command to install the GNOME 3.25 runtime and SDK is::

  $ flatpak install flathub org.gnome.Platform//3.26 org.gnome.Sdk//3.26

A number of the examples in these docs use this runtime and SDK, so it is a good idea to try this command yourself.
