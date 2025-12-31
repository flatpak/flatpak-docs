Available Runtimes
==================

This page provides information about available Flatpak runtimes. It is
primarily intended as information for application developers and
distributors.

There are currently three main runtimes available: Freedesktop, GNOME and
KDE all of which are hosted on `Flathub <https://flathub.org/>`_. Each
runtime comes with a parent SDK for building, a Docs, a Debug and a
Locale extension to provide documentation, debug symbols and localisation
support, respectively and other extensions or extension points for
specific uses.

Freedesktop
-----------

The `Freedesktop runtime <https://gitlab.com/freedesktop-sdk/freedesktop-sdk/>`_
is the standard runtime that can be used for any application. It
contains a set of essential libraries, provides the graphics and the
toolchain stack and forms the base of the GNOME and KDE runtimes.

It also provides a base set of extensions and extension points used
by other runtimes and applications. See
:ref:`extension:Extensions or extension points defined by runtime` for
more details.

A given branch of the Freedesktop runtime has a 2 year support period
after which they are declared EOL. A new major version is published on
August of every year. The release schedule can be found on `Gitlab <https://gitlab.com/freedesktop-sdk/freedesktop-sdk/-/wikis/Releases>`_
and announcements of new major version releases are made on `Flathub Discourse <https://discourse.flathub.org/c/announcements/6>`_.

The Freedesktop platform (``org.freedesktop.Platform`` and
``org.freedesktop.Sdk``) provides strict ABI/API stability in a
major version.

====================================================== =====================================
ID                                                     Description
====================================================== =====================================
org.freedesktop.Platform                               Runtime
org.freedesktop.Sdk                                    SDK
org.freedesktop.Platform.Locale                        Runtime translations (extension)
org.freedesktop.Sdk.Debug                              SDK debug information (extension)
org.freedesktop.Sdk.Locale                             SDK translations (extension)
org.freedesktop.Sdk.Docs                               SDK documentation (extension)
====================================================== =====================================

GNOME
-----

The `GNOME runtime <https://gitlab.gnome.org/GNOME/gnome-build-meta>`_
is appropriate for any application that uses the GNOME platform. It is
based on the Freedesktop runtime and adds the  libraries and components
used by the GNOME platform.

Major version releases of the runtime are synced with `GNOME releases <https://release.gnome.org/calendar/>`_
and are announced on `GNOME Discourse <https://discourse.gnome.org/tag/announcement>`_.
Usually a given branch of the runtime is supported for a year and EOL-ed
upon the release of a newstable version.

====================================================== =====================================
ID                                                     Description
====================================================== =====================================
org.gnome.Platform                                     Runtime
org.gnome.Sdk                                          SDK
org.gnome.Platform.Locale                              Runtime translations (extension)
org.gnome.Sdk.Debug                                    SDK debug information (extension)
org.gnome.Sdk.Locale                                   SDK translations (extension)
org.gnome.Sdk.Docs                                     SDK documentation (extension)
====================================================== =====================================

KDE
---

The `KDE runtime <https://invent.kde.org/packaging/flatpak-kde-runtime>`_
is also based on the Freedesktop runtime and adds Qt and KDE Frameworks.
It is appropriate for any application that makes use of the KDE
platform and most Qt-based applications.

Qt5 LTS (based on KDE's Qt patch collection) branches are created on
Freedesktop runtime major version releases and Qt6 branches are created
on new Qt6 releases. More information about the support policy can be
found on the `KDE wiki <https://community.kde.org/Policies/Flatpak_Runtime_Update_Policy>`_
and announcements for new runtime branch releases are made on the
`KDE Discourse <https://discuss.kde.org/c/announcement/9>`_.

====================================================== =====================================
ID                                                     Description
====================================================== =====================================
org.kde.Platform                                       Runtime
org.kde.Sdk                                            SDK
org.kde.Platform.Locale                                Runtime translations (extension)
org.kde.Sdk.Debug                                      SDK debug information (extension)
org.kde.Sdk.Locale                                     SDK translations (extension)
org.kde.Sdk.Docs                                       SDK documentation (extension)
====================================================== =====================================

elementary
----------

The `elementary runtime <https://github.com/elementary/flatpak-platform>`_
is hosted by elementary and is appropriate for any application that
would like to publish to the elementary AppCenter. It is based on the
GNOME runtime and adds the elementary platform.

====================================================== =====================================
ID                                                     Description
====================================================== =====================================
io.elementary.Platform                                 Runtime
io.elementary.Sdk                                      SDK
io.elementary.Platform.Locale                          Runtime translations (extension)
io.elementary.Sdk.Debug                                SDK debug information (extension)
io.elementary.Sdk.Locale                               SDK translations (extension)
io.elementary.Sdk.Docs                                 SDK documentation (extension)
====================================================== =====================================

Check software available in runtimes
------------------------------------

The best way to check software packaged in the runtime is to look at
their respective manifests in the git repository or release contents.

The latter can be done using::

	flatpak run --command=cat <ID>//<branch> /usr/manifest.json|jq -r '."modules"|.[]|."name"'|sed -E 's#.*/(.*)\.bst#\1#'|sort -u

	# List all contents in GNOME 47 runtime
	flatpak run --command=cat org.gnome.Platform//47 /usr/manifest.json|jq -r '."modules"|.[]|."name"'|sed -E 's#.*/(.*)\.bst#\1#'|sort -u

``pkg-config`` can be used for modules that install pkg-config files::

	flatpak run --command=pkg-config <SDK ID>//<branch> --list-all

	# Check version appstream in Freedesktop SDK 24.08
	flatpak run --command=pkg-config org.freedesktop.Sdk//24.08 --modversion appstream

``ldconfig`` can be used to list all libraries::

	flatpak run --command=ldconfig <ID>//<branch> -p

	# Check libraries in Freedesktop runtime 24.08
	flatpak run --command=ldconfig org.freedesktop.Platform//24.08 -p|awk '/\.so/ {print $1}'
