Dotnet
======

Prerequisites
~~~~~~~~~~~~~
  - The application is built with a Linux-compatible .NET desktop application framework compatible such as `GTKSharp <https://github.com/GtkSharp/GtkSharp>`__, `Avalonia UI <https://www.avaloniaui.net/>`__, or `Eto <https://github.com/picoe/Eto>`__
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

3. Install the FreeDesktop SDKs (When prompted for a version, select the
   latest available version)

  .. code-block:: shell

        flatpak install flathub org.freedesktop.Platform org.freedesktop.Sdk

4. Install the Flatpak SDK extension for your dotnet version
   (e.g. `dotnet6 <https://github.com/flathub/org.freedesktop.Sdk.Extension.dotnet6>`__, `dotnet7 <https://github.com/flathub/org.freedesktop.Sdk.Extension.dotnet7>`__)

  .. code-block:: shell

        flatpak install org.freedesktop.Sdk.Extension.dotnet6


Creating the Flatpak
^^^^^^^^^^^^^^^^^^^^

.. note::

  Here is a brief description of the placeholders in the below example:

  - ``<namespace>``: The prefix that goes before the name of your Flatpak, which uniquely identifies the author. See `"Conventions - Application IDs" section <https://docs.flatpak.org/en/latest/conventions.html#application-ids>`__ to find the appropriate value for your use case.
  - ``<app-name>``: The name of your application to use for Flatpak, usually somewhat similar to the project name inside the solution
  - ``<project-name>``: The name of your ``.csproj`` file
  - ``<user-name>``: The username you use in your Git server (GitHub, GitLab, Bitbucket)
  - ``<release-number>``: The numbered release to build from, in the form of a Git `Tag <https://git-scm.com/book/en/v2/Git-Basics-Tagging>`__.

5.  Create a new folder somewhere different from your existing project


6.  Create a YAML file titled
    ``<namespace>.<app-name>.yaml`` with the following
    example template, replacing the placeholders with the appropriate
    information: \

  .. code-block:: yaml

      app-id: <namespace>.<app-name>
      runtime: org.freedesktop.Platform
      runtime-version: '22.08'
      sdk: org.freedesktop.Sdk
      sdk-extensions:
        - org.freedesktop.Sdk.Extension.dotnet6
      build-options:
        prepend-path: "/usr/lib/sdk/dotnet6/bin"
        append-ld-library-path: "/usr/lib/sdk/dotnet6/lib"
        prepend-pkg-config-path: "/app/lib/pkgconfig:/app/share/pkgconfig:/usr/lib/pkgconfig:/usr/share/pkgconfig:/usr/lib/sdk/dotnet6/lib/pkgconfig"

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
          - /usr/lib/sdk/dotnet6/bin/install.sh

        - name: <app-name>
          buildsystem: simple
          sources:
            - type: git
              url: https://github.com/<username>/<project-name>.git
              tag: <release-number>
            - ./nuget-sources.json
          build-commands:
            - dotnet publish <project-name>/<project-name>.csproj -c Release --no-self-contained --source ./nuget-sources
            - mkdir -p ${FLATPAK_DEST}/bin
            - cp -r <project-name>/bin/Release/net6.0/publish/* ${FLATPAK_DEST}/bin

  .. note::

      For providing access to other things such as the network or
      filesystem, see the `“Sandbox Permissions” section <https://docs.flatpak.org/en/latest/sandbox-permissions.html>`__

7.  Copy and save the dotnet NuGet sources generator script
    ``flatpak-dotnet-generator.py`` from the `Flatpak Builder Tools
    repository <https://github.com/flatpak/flatpak-builder-tools>`__, to
    the current folder, or run the following command to download it:

  .. code-block:: shell

        wget https://raw.githubusercontent.com/flatpak/flatpak-builder-tools/master/dotnet/flatpak-dotnet-generator.py

8.  Clone down your project repository to the folder

  .. code-block:: shell

        git clone https://github.com/<username>/<app-name>.git

9.  Run the NuGet source config generator script ``flatpak-dotnet-generator.py`` with the following arguments:

  .. code-block:: shell

        python3 flatpak-dotnet-generator.py --dotnet 6 nuget-sources.json <app-name>/<project-name>/<project-name>.csproj

10. Run the Flatpak Builder script to build the local Flatpak

  .. code-block:: shell

        flatpak-builder build-dir <namespace>.<app-name>.yaml --force-clean


Testing the build
^^^^^^^^^^^^^^^^^

11. Rebuild and install the local flatpak

  .. code-block:: shell

        flatpak-builder --user --install build-dir <namespace>.<app-name>.yaml --force-clean


12. Run the installed Flatpak application

  .. code-block:: shell

        flatpak run <namespace>.<app-name>

