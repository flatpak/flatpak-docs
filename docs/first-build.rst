Building your first Flatpak
===========================

.. note::

  App stores like `Flathub <https://docs.flathub.org/docs/for-app-authors/submission>`_
  have additional requirements on the building process. Please refer to
  their respective documentation for details.

This tutorial provides a quick introduction to build, install and share
a basic flatpak package.

In order to complete this tutorial, please install ``flatpak``. The
`setup page <https://flatpak.org/setup/>`_ covers steps for most of the
popular distributions.

Please also install ``flatpak-builder`` from your distribution. There is
also a flatpak-builder `Flatpak package <https://flathub.org/apps/org.flatpak.Builder>`_
on Flathub (this may contain Flathub specific downstream modifications).

Then install the Flathub repository `userwide`::

  flatpak remote-add --if-not-exists --user flathub https://dl.flathub.org/repo/flathub.flatpakrepo

1. Runtime and SDK
------------------

Flatpak requires every app to specify a runtime for its basic runtime
dependencies and a matching SDK which is a superset of the runtime
containing additional development tools, libraries and headers.

In this tutorial we will use the Freedesktop 25.08 runtime and SDK from
the Flathub repository.

2. Add a manifest
-----------------

Each Flatpak is built using a manifest file which provides basic information
about the application and build instructions.

To create a manifest for our hello world app, paste the following into
an empty file and save it as ``org.flatpak.Hello.yml``.

.. code-block:: yaml

  id: org.flatpak.Hello
  runtime: org.freedesktop.Platform
  runtime-version: '25.08'
  sdk: org.freedesktop.Sdk
  command: hello
  modules:
    - name: hello
      buildsystem: simple
      build-commands:
        - install -Dm755 hello.sh /app/bin/hello
      sources:
        - type: script
          dest-filename: hello.sh
          commands:
            - echo "Hello world, from a sandbox"

The `application` here is a simple script, that is self contained in the
manifest! The `install -Dm755 <https://www.gnu.org/software/coreutils/manual/html_node/install-invocation.html>`_ command will create a file containing our hello world script in ``/app/bin/hello`` with `execution permissions <https://www.gnu.org/software/coreutils/manual/html_node/Numeric-Modes.html>`_ inside the sandbox.

In a more complex application, the manifest would list multiple
modules and build instructions.

3. Build and install
--------------------

Now that the app has a manifest, ``flatpak-builder`` can be used to build
and install it::

  flatpak-builder --force-clean --user --install-deps-from=flathub --repo=repo --install builddir org.flatpak.Hello.yml

This command will first build each module that is listed in the manifest,
install the contents of ``builddir/files`` in ``/app`` inside the sandbox
and finally install the the whole application as a flatpak.

4. Run the application
----------------------

To run the application::

  flatpak run org.flatpak.Hello

If you see ``Hello world, from a sandbox`` on the terminal then
congratulations, you've made an app!

5. Share the application
------------------------

If you want to share the application you can create a single-file bundle.
See :doc:`single-file-bundles` for more details on it.

.. code-block:: bash

  flatpak build-bundle repo hello.flatpak org.flatpak.Hello --runtime-repo=https://flathub.org/repo/flathub.flatpakrepo

Now you can send the ``hello.flatpak`` file to someone and if they have
the Flathub repository set up and a working network connection to install
the runtime, they can install it with::

  flatpak install --user hello.flatpak

.. tip::

  To uninstall, run ``flatpak remove org.flatpak.Hello``
