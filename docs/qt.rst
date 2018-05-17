Qt
==

For Qt-based applications we have the `org.kde.Sdk` that will offer us most Qt
modules and KDE Frameworks for our applications to use.

For example, the following JSON makes building a random Qt application really
straight-forward.

.. code-block:: json
{
    "app-id": "org.flatpak.qtdemo",
    "runtime": "org.kde.Platform",
    "runtime-version": "5.11",
    "sdk": "org.kde.Sdk",
    "command": "flatpak-demo",
    "finish-args": [
        "--share=ipc",
        "--socket=x11",
        "--socket=wayland",
        "--filesystem=host",
        "--device=dri"
    ],
    "modules": [
        {
            "name": "flatpak-demo",
            "buildsystem": "cmake-ninja",
            "config-opts": ["-DCMAKE_BUILD_TYPE=RelWithDebInfo"],
            "sources": [
                {
                    "type": "archive",
                    "url": "https://github.com/flatpak/qt-flatpak-demo/archive/v1.1.2.tar.gz",
                    "sha256": "1a1cc5d0f06ad949d6854c772ec9624b8856a0a4f880355f51058bc0dd52ba7a"
                }
            ]
        }
    ]
}

Contents
========

The org.kde.Platform runtime includes all Qt to use, including some KDE Frameworks. If there's any issues we encourage you to report the issues to be addressed. If this is not enough, it's also possible to use the org.freedesktop.Platform as a base and bundle the parts of Qt you need.

