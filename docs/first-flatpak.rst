Building Your First Flatpak
===========================

This tutorial provides a quick introduction to Flatpak. In it, you will learn
how to create a basic Flatpak application, which can be installed and run.

To complete this tutorial, you should have installed Flatpak by following
`the setup guide <https://flatpak.org/getting.html>`_. You will also need to
haves the ``flatpak-builder`` tool, which is generally available as a package.

1. Install a runtime and the matching SDK
-----------------------------------------

Flatpak requires every app to specify a runtime that it uses for its basic
dependencies. Each runtime has a matching SDK (Software Development Kit), which
contains all the things that are in the runtime, plus headers and development
tools (similar to what is typically found in -devel/-dev packages in Linux
distributions). This SDK is required to build apps for the runtime.

In this tutorial we will use the Freedesktop runtime version 1.6. This runtime
is provided by the Flathub repository. To add this, run::

  $ flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo

Then, to install the runtime and the SDK, run::

  $ flatpak install flathub org.freedesktop.Platform//1.6 org.freedesktop.Sdk//1.6

2. Create the app
-----------------

The app that is going to be created for this tutorial is a a simple script. To
create it, copy the following to an empty file and save it as `hello.sh`::

  #!/bin/sh
  echo "Hello world, from a sandbox"

3. Add a manifest
-----------------

Most Flatpaks are built using the `flatpak-builder` tool. This reads a manifest
file which describes the key properties of the application and how it is to be
built.

To add a manifest to the hello world app, add the following to an empty file:

.. code-block:: json

  {
      "app-id": "org.flatpak.Hello",
      "runtime": "org.freedesktop.Platform",
      "runtime-version": "1.6",
      "sdk": "org.freedesktop.Sdk",
      "command": "hello.sh",
      "modules": [
          {
              "name": "hello",
              "buildsystem": "simple",
              "build-commands": [
                  "install -D hello.sh /app/bin/hello.sh"
              ],
              "sources": [
                  {
                      "type": "file",
                      "path": "hello.sh"
                  }
              ]
          }
      ]
  }

Now save the file alongside `hello.sh` and call it `org.flatpak.Hello.json`.

In a more complex application, the manifest would list multiple modules. The
last one would typically be the application itself, and the earlier ones would
be dependencies that are bundled with the app because they are not part of the
runtime.

4. Build the application
------------------------

Now that the app has a manifest, `flatpak-builder` can be used to build it.
This is done by specifying the the manifest file and a target directory::

  $ flatpak-builder app-dir org.flatpak.Hello.json

This command will build each module that is listed in the manifest and install
it to the `/app` subdirectory, inside the `app-dir` directory.

5. Test the build
-----------------

To verify that the build was successful, the following can be run::

  $ flatpak-builder --run app-dir org.flatpak.Hello.json hello.sh

Congratulations, you've made an app!

6. Put the app in a repository
------------------------------

Before you can install and run the app, it first needs to be put in a
repository. This is done by passing the `--repo` argument to `flatpak-builder`::

 $ flatpak-builder --repo=repo --force-clean app-dir org.flatpak.Hello.json

This does the build again, and at the end exports the result to a local
directory called `repo`. Note that `flatpak-builder` keeps a cache of previous
builds in the `.flatpak-builder` subdirectory, so doing a second build like
this is very fast.

This second time we passed in `--force-clean`, which means that the previously
created `app-dir` directory was deleted before the new build was started.

7. Install the app
------------------

Now we're ready to add the repository that was just created and install the
app. This is done with two commands::

  $ flatpak --user remote-add --no-gpg-verify tutorial-repo repo
  $ flatpak --user install tutorial-repo org.flatpak.Hello

The first command adds the repository that was created in the previous step.
The second command installs the app from the repository.

Both these commands use the `--user` argument, which means that the repository
and the app are added per-user rather than system-wide. This is useful for testing.

Note that the repository was added with `--no-gpg-verify`, since a GPG key
wasn't specified when the app was built. This is fine for testing, but for
official repositories you should sign them with a private GPG key.

8. Run the app
--------------

All that's left is to try the app. This can be done with the following command::

  $ flatpak run org.flatpak.Hello

This runs the app, so that it prints `Hello world, from a sandbox`.
