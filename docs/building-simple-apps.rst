Building Simple Apps
====================

The ``flatpak`` utility provides a simple set of commands for building and distributing applications. These allow creating new Flatpaks, into which new or existing applications can be built. This section describes how to build a simple application which doesn't require any additional dependencies outside of the runtime it is built against.

Installing an SDK
-----------------

As described above, an SDK is a special type of runtime that is used to build applcations. Typically, an SDK is paired with a runtime that will be used by the app at runtime. For example the GNOME 3.22 SDK is used to build applications that use the GNOME 3.22 runtime. The rest of this guide uses this SDK and runtime for its examples. To do this, download the repository GPG key and then add the repository that contains the runtime and SDK::
  
  $ flatpak remote-add --from gnome https://sdk.gnome.org/gnome.flatpakrepo

You can now download and install the runtime and SDK. (If you have already completed the tutorial on the Flatpak homepage, you will already have the runtime installed.) ::

  $ flatpak install gnome org.gnome.Platform//3.22 org.gnome.Sdk//3.22
 
This might be a good time to try installing an application and having a look 'under the hood'. To do this, you need to add a repository that contains applications. In this case we are going to use the gnome-apps repository and install gedit::

  $ flatpak remote-add --from gnome-apps https://sdk.gnome.org/gnome-apps.flatpakrepo
  $ flatpak install gnome-apps org.gnome.gedit

You can now use the following command to get a shell in the 'devel sandbox'::

  $ flatpak run --devel --command=bash org.gnome.gedit

This gives you an environment which has the application bundle mounted in ``/app``, and the SDK it was built against mounted in ``/usr``. You can explore these two directories to see what a typical flatpak looks like, as well as what is included in the SDK.

Creating an app
---------------

To create an application, the first step is to use the build-init command. This creates a directory into which an applcation can be built, which contains the correct directory structure and a metadata file which contains information about the app. The format for build-init is::

  $ flatpak build-init DIRECTORY APPNAME SDK RUNTIME [BRANCH]

* DIRECTORY is the name of the directory that will be created to contain the application
* APPNAME is the D-Bus name of the application
* SDK is the name of the SDK that will be used to build the application
* RUNTIME is the name of the runtime that will be required by the application
* BRANCH is typically the version of the SDK and runtime that will be used

For example, to build the GNOME Dictionary application using the GNOME 3.22 SDK, the command would look like::

  $ flatpak build-init dictionary org.gnome.Dictionary org.gnome.Sdk org.gnome.Platform 3.22

You can try this command now. In the next step we will build an application inside the resulting dictionary directory.

Building
--------

``flatpak build`` is used to build an application using an SDK. This command is used to provide access to a sandbox. For example, the following will create a file inside the appdir sandbox (in the files directory)::

  $ flatpak build dictionary touch /app/some_file
  
(It is best to remove this file before continuing.)

The build command allows existing applications that have been made using the traditional configure, make, make install routine to be built inside a flatpak. You can try this using GNOME Dictionary. First, download the source files, extract them and switch to the resulting directory::

  $ wget https://download.gnome.org/sources/gnome-dictionary/3.22/gnome-dictionary-3.22.0.tar.xz
  $ tar xvf gnome-dictionary-3.22.0.tar.xz
  $ cd gnome-dictionary-3.22.0/

Then you can use the build command to build and install the source inside the dictionary directory that was previously made::

  $ flatpak build ../dictionary ./configure --prefix=/app
  $ flatpak build ../dictionary make
  $ flatpak build ../dictionary make install
  $ cd ..

Since these are run in a sandbox, the compiler and other tools from the SDK are used to build and install, rather than those on the host system.

Completing the build
--------------------

Once an application has been built, the ``build-finish`` command needs to be used to specify access to different parts of the host, such as networking and graphics sockets. This command is also used to specify the command that is used to run the app (done by modifying the metadata file), and to create the application's exports directory. For example::

  $ flatpak build-finish dictionary --socket=x11 --share=network --command=gnome-dictionary
  
At this point you have successfully built a flatpak and prepared it to be run. To test the app, you need to export the Dictionary to a repository, add that repository and then install and run the app::

  $ flatpak build-export repo dictionary
  $ flatpak --user remote-add --no-gpg-verify --if-not-exists tutorial-repo repo
  $ flatpak --user install tutorial-repo org.gnome.Dictionary
  $ flatpak run org.gnome.Dictionary

This exports the app, creates a repository called tutorial-repo, installs the Dictionary application in the per-user installation area and runs it.

