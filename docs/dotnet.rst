Dotnet
======

Prerequisites
~~~~~~~~~~~~~
- The application is built with a Linux-compatible .NET desktop application framework such as:
    - `Avalonia UI <https://avaloniaui.net/>`__
    - `Uno <https://platform.uno/>`__
    - `Eto <https://github.com/picoe/Eto>`__
    - `GTKSharp <https://github.com/GtkSharp/GtkSharp>`__
- The application's source code is hosted on a Git server such as GitHub, GitLab, or Bitbucket
- A Git commit with a `Tag <https://git-scm.com/book/en/v2/Git-Basics-Tagging>`__ exists on the repository to use for the build. This usually represents the version number of the application (e.g. v1.0, v1.1, etc.).

Steps for Packaging
~~~~~~~~~~~~~~~~~~~

Installing dependencies
^^^^^^^^^^^^^^^^^^^^^^^

1. Install Flatpak using the method provided for your distribution
   `Flatpak - Quick Setup <https://flatpak.org/setup/>`__

2. Install Flatpak Builder as both a flatpak package and a distribution package (apt / dnf)

.. code-block:: shell

  flatpak install org.flatpak.Builder
  sudo apt install flatpak-builder


Creating the Flatpak
^^^^^^^^^^^^^^^^^^^^

.. note::

  Here is a brief description of the placeholders in the below example:

  - ``<app-id>``: The name of your Flatpak, see `"Conventions - Application IDs" section <https://docs.flatpak.org/en/latest/conventions.html#application-ids>`__.
  - ``<app-name>``: The name of the root folder of your app repository
  - ``<project-name>``: The name of your ``.csproj`` file
  - ``<git-server-url>``: The URL to your git server (e.g. ``https://github.com/``, ``https://gitlab.com``)
  - ``<user-name>``: The username you use in your Git server (GitHub, GitLab, Bitbucket)
  - ``<release-number>``: The numbered release to build from, in the form of a Git `Tag <https://git-scm.com/book/en/v2/Git-Basics-Tagging>`__.

3.  Create a new folder somewhere different from your existing project

4.  Create a YAML file titled ``<app-id>.yaml`` with the following example template, replacing the placeholders with the appropriate information. (Note: If your project file lives in a subfolder, be sure to include that in the build paths in this file and subsequent commands as well.): 

.. code-block:: yaml

  app-id: <app-id>
  runtime: org.freedesktop.Platform
  runtime-version: '23.08'
  sdk: org.freedesktop.Sdk
  sdk-extensions:
    - org.freedesktop.Sdk.Extension.dotnet8
  build-options:
    prepend-path: "/usr/lib/sdk/dotnet8/bin"
    append-ld-library-path: "/usr/lib/sdk/dotnet8/lib"
    prepend-pkg-config-path: "/app/lib/pkgconfig:/app/share/pkgconfig:/usr/lib/pkgconfig:/usr/share/pkgconfig:/usr/lib/sdk/dotnet8/lib/pkgconfig"

  command: <project-name>

  finish-args:
    - --device=dri
    # TODO: Replace this with wayland and fallback-x11 once Wayland support
    #       becomes available:
    #       https://github.com/AvaloniaUI/Avalonia/pull/8003
    - --socket=x11
    - --share=ipc
    - --env=DOTNET_ROOT=/app/lib/dotnet

  modules:
    - name: dotnet
      buildsystem: simple
      build-commands:
      - /usr/lib/sdk/dotnet8/bin/install.sh

    - name: <app-id>
      buildsystem: simple
      sources:
        - type: git
          url: <git-server-url>/<user-name>/<project-name>.git
          tag: <release-number>
        - ./nuget-sources.json
      build-commands:
        - dotnet publish <project-name>.csproj -c Release --no-self-contained --source ./nuget-sources
        - mkdir -p ${FLATPAK_DEST}/bin
        - cp -r /bin/Release/net8.0/publish/* ${FLATPAK_DEST}/bin

.. note::

    For providing access to other things such as the network or
    filesystem, see the `“Sandbox Permissions” section <https://docs.flatpak.org/en/latest/sandbox-permissions.html>`__

5.  Copy and save the dotnet NuGet sources generator script
    ``flatpak-dotnet-generator.py`` from the `Flatpak Builder Tools
    repository <https://github.com/flatpak/flatpak-builder-tools>`__, to
    the current folder, or run the following command to download it:

.. code-block:: shell

      wget https://raw.githubusercontent.com/flatpak/flatpak-builder-tools/master/dotnet/flatpak-dotnet-generator.py

6.  Clone down your project repository to the folder

.. code-block:: shell

      git clone <git-server-url>/<username>/<app-name>.git

7.  Run the NuGet source config generator script ``flatpak-dotnet-generator.py`` with the following arguments:

.. code-block:: shell

      python3 flatpak-dotnet-generator.py --dotnet 8 nuget-sources.json <app-name>/<project-name>.csproj

8. Run the Flatpak Builder script to build and install the local Flatpak

.. code-block:: shell

      flatpak-builder build-dir --install-deps-from=flathub --user --force-clean --install --repo=repo <app-id>.yaml


Testing the build
^^^^^^^^^^^^^^^^^

9. Run the installed Flatpak application

.. code-block:: shell

      flatpak run <app-id>

