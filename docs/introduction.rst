Introduction to Flatpak
=======================

Key concepts
------------

Flatpak is best understood through its key concepts: runtimes, bundled libraries, SDKs and sandboxes. These help to explain how Flatpak differs from traditional application distribution on Linux, as well as the framework's capabilities.

.. image:: ../images/diagram.svg

Runtimes
^^^^^^^^

Runtimes provide the environment that each application runs in, including the basic dependencies they might require. Each runtime can be thought of as a ``/usr`` filesystem (indeed, when an app is run, its runtime is mounted at ``/usr``). Various runtimes are available, from more minimal (but more stable) Freedesktop runtimes, to larger runtimes produced by desktops like GNOME or KDE. (The `runtimes page <http://flatpak.org/runtimes.html>`_ provides an overview of the runtimes that are currently available.)

Each application must be built against a runtime, and this runtime must be installed on a host system in order for the application to run. Users can install multiple different runtimes at the same time, including different versions of the same runtime.

Flatpak identifies runtimes (as well as SDKs and applications) by a triple of name/arch/branch. The name is expected to be in inverse-dns notation, which needs to match the D-Bus name used for the application. For example: ``org.gnome.Sdk/x86_64/3.14`` or ``org.gnome.Builder/i386/master``.

Bundled libraries
^^^^^^^^^^^^^^^^^

If an application requires any dependencies that aren't in its runtime, they can be bundled along with the application itself. This allows apps to use dependencies that aren't available in a distribution, or to use a different version of a dependency from the one that's installed on the host.

Both runtimes and app bundles can be installed per-user and system-wide.

SDKs (Software Developer Kits)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

An SDK is a runtime that includes the 'devel' parts which are not needed at runtime, such as build and packaging tools, header files, compilers and debuggers. Each application is built against an SDK, which is typically paired with a runtime (this is the runtime that will be used by the application at runtime).

Sandboxes
^^^^^^^^^

With Flatpak, each app is built and run in an isolated environment. By default, the application can only 'see' itself and its runtime. Access to user files, network, graphics sockets, subsystems on the bus and devices have to be explicitly granted. (As will be described later, Flatpak provides several ways to do this.) Access to other things, such as other processes, is (deliberately) not possible.

Technologies
------------

Flatpak tries to avoid reinventing the wheel. We build on existing technologies where it makes sense. Many of the important ingredients for Flatpak are inherited from Linux containers and related initiatives:

* The `bubblewrap <https://github.com/projectatomic/bubblewrap>`_ utility from `Project Atomic <http://www.projectatomic.io/>`_, which lets unprivileged users set up and run containers, using kernel features such as:

  * Cgroups
  * Namespaces
  * Bind mounts
  * Seccomp rules

* `systemd <https://www.freedesktop.org/wiki/Software/systemd/>`_ to set up cgroups for our sandbox
* `D-Bus <https://www.freedesktop.org/wiki/Software/dbus/>`_, a well-established way to provide high-level APIs to application
* The OCI format from the `Open Container Initiative <https://www.opencontainers.org/>`_, as a convenient transport format for single-file bundles
* The `OSTree <https://ostree.readthedocs.io/en/latest/>`_ system for versioning and distributing filesystem trees
* `Appstream <https://www.freedesktop.org/software/appstream/docs/>`_ metadata that makes Flatpak apps show up nicely in software-center applications

The flatpak command
--------------------

``flatpak`` is the command that is used to install, remove and update runtimes and applications. It can also be used to view what is currently installed, and has commands for building and distributing application bundles. ``flatpak --help`` provides a full list of available commands.

Most flatpak commands are performed system-wide by default. To perform a command for the current user only, use the ``--user`` option.


