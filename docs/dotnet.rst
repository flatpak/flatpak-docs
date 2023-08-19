Dotnet
======

Prerequisites
~~~~~~~~~~~~~
   -  Is written with a Linux-compatible .NET desktop application framework compatible such as
      `GTKSharp <https://github.com/GtkSharp/GtkSharp>`__, `Avalonia
      UI <https://www.avaloniaui.net/>`__, or
      `Eto <https://github.com/picoe/Eto>`__
   -  Has source code hosted on a Git server such as GitHub, GitLab,
      or Bitbucket

Steps for Packaging
~~~~~~~~~~~~~~~~~~~

.. note::

  The steps below make a couple of assumptions for the sake of example, that may require modification to work for your use case:

  - The application is namespaced with “com.github.[[username]]” for GitHub users.
    (This can be replaced with any domain that you own, such as another Git server or your own website domain.)
  - The application uses .NET 7.
    (It still should be applicable for other versions by modifying the version number)

Installing dependencies
^^^^^^^^^^^^^^^^^^^^^^^

1. Install Flatpak using the method provided for your distribution
   `Flatpak - Quick Setup <https://flatpak.org/setup/>`__
2. Install the FreeDesktop SDKs (When prompted for a version, select the
   latest available version)

  .. code-block:: shell

        flatpak install flathub org.freedesktop.Platform org.freedesktop.Sdk

3. Install the Flatpak SDK extension for your dotnet version
   (e.g. `dotnet6 <https://github.com/flathub/org.freedesktop.Sdk.Extension.dotnet6>`__, `dotnet7 <https://github.com/flathub/org.freedesktop.Sdk.Extension.dotnet7>`__)

  .. code-block:: shell

        flatpak install org.freedesktop.Sdk.Extension.dotnet7


Creating the Flatpak
^^^^^^^^^^^^^^^^^^^^

5.  Create a new folder somewhere different from your existing project

6.  Create a YAML file titled
    ``com.github.[[username]].[[project-name]].yaml`` with the following
    example template, replacing the placeholders with the appropriate
    information: \

  .. code-block:: yaml

      app-id: com.github.[[username]].[[project-name]]
      runtime: org.freedesktop.Platform
      runtime-version: '22.08'
      sdk: org.freedesktop.Sdk
      sdk-extensions:
        - org.freedesktop.Sdk.Extension.dotnet7
      build-options:
        prepend-path: "/usr/lib/sdk/dotnet7/bin"
        append-ld-library-path: "/usr/lib/sdk/dotnet7/lib"
        env:
          PKG_CONFIG_PATH: "/app/lib/pkgconfig:/app/share/pkgconfig:/usr/lib/pkgconfig:/usr/share/pkgconfig:/usr/lib/sdk/dotnet7/lib/pkgconfig"

      command: [[project-name]]

      finish-args:
        - --device=dri
        # TODO: Replace this with wayland and fallback-x11 once Wayland support
        #       becomes available:
        #       https://github.com/AvaloniaUI/Avalonia/pull/8003
        - --socket=x11
        - --env=DOTNET_ROOT=/app/lib/dotnet

      modules:
        - name: dotnet
          buildsystem: simple
          build-commands:
          - /usr/lib/sdk/dotnet7/bin/install.sh

        - name: [[project-name]]
          buildsystem: simple
          sources:
            - type: git
              url: https://github.com/[[username]]/[[project-name]].git
            - ./nuget-sources.json
          build-commands:
            - dotnet publish [[project-name]]/[[project-name]].csproj -c Release --no-self-contained --source ./nuget-sources
            - mkdir -p ${FLATPAK_DEST}/bin
            - cp -r ${FLATPAK_BUILDER_BUILDDIR}/[[project-name]]/bin/Release/net7.0/publish/* ${FLATPAK_DEST}/bin

  .. note::

      For providing access to other things such as the network or
      filesystem, see the `“Sandbox Permissions” section <../sandbox-permissions.html>`__

7.  Copy and save the dotnet NuGet sources generator script
    ``flatpak-dotnet-generator.py`` from the `Flatpak Builder Tools
    repository <https://github.com/flatpak/flatpak-builder-tools>`__, to
    the current folder, or run the following command to download it:

  .. code-block:: shell

        wget https://raw.githubusercontent.com/flatpak/flatpak-builder-tools/master/dotnet/flatpak-dotnet-generator.py

8.  Clone down your project repository to the folder

  .. code-block:: shell

        git clone https://github.com/[[username]]/[[project]].git

9.  Run the NuGet source config generator script
    ``flatpak-dotnet-generator.py`` with the following arguments:

  .. code-block:: shell

        python3 flatpak-dotnet-generator.py --dotnet 7 nuget-sources.json [[project-name]]/[[project-name]]/[[project-name]].csproj

10. Run the Flatpak Builder script to build the local Flatpak

  .. code-block:: shell

        flatpak-builder build-dir ./org.[[username]].[[project-name]].yaml --force-clean

11. If the above build ran successfully, install the local flatpak

  .. code-block:: shell

        flatpak-builder --user --install build-dir ./org.[[username]].[[project-name]].yaml --force-clean


12. Run the newly generated and installed Flatpak application

  .. code-block:: shell

        flatpak run com.github.[[username]].[[project]]

