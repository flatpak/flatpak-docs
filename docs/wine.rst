Wine
====

Flathub contains an unofficial Wine stable BaseApp, stylized as `org.winehq.Wine <https://github.com/flathub/org.winehq.Wine>`__, to have and run Wine directly from within the Flatpak application. To use the Wine BaseApp, we need to explicitly specify it in the manifest and inherit all needed extensions:

.. code-block:: yaml

  base: org.winehq.Wine
  base-version: stable-21.08 # Version of Wine BaseApp
  inherit-extensions:
    - org.freedesktop.Platform.GL32
    - org.freedesktop.Platform.ffmpeg-full
    - org.freedesktop.Platform.ffmpeg_full.i386
    - org.winehq.Wine.gecko
    - org.winehq.Wine.mono
    - org.winehq.Wine.DLLs
