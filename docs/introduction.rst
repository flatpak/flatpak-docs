Introduction to Flatpak
=======================

Flatpak is a technology for building, distributing, installing and running applications. It is primarily targeted at the Linux desktop, although it can also be used as the basis for application distribution in other contexts, such as embedded systems.

Flatpak has been designed and implemented with a number of goals:

* Allow the same application build to be installed on any Linux distribution.
* Provide consistent environments for applications, to facilitate testing and reduce bugs.
* Ensure forward compatibility of applications, by allowing multiple versions of runtimes to be simultaneously installed.
* Allow applications to easily use libraries that aren't available in Linux distributions (or aren't consistently available).
* Increase the security of Linux desktops, by isolating applications in sandboxes.

General information about Flatpak can be found on `flatpak.org <http://flatpak.org/>`_.

Basic concepts
--------------

Flatpak can be understood through a small number of key concepts.

.. image:: ../images/diagram.svg

Runtimes
^^^^^^^^

Runtimes provide the basic dependencies that are used by applications. Each application must be built against a runtime, and this runtime must be installed on a host system in order for the application to run (Flatpak can automatically install the runtime required by an application). Multiple different runtimes can be installed at the same time, including different versions of the same runtime.

A small number of runtimes are available, including the minimal and stable Freedesktop runtimes, as well as runtimes which contain the GNOME and KDE stacks. (See :doc:`available-runtimes` for an overview of the runtimes that are currently available.)

Runtimes are distribution agnostic and do not depend on particular distribution versions. This means that they provide a stable, cross-distribution base for applications, and allow applications to continue to work irrespective of operating system updates.

Bundled libraries
^^^^^^^^^^^^^^^^^

If an application requires any dependencies that aren't in its runtime, they can be bundled as part of the application. This gives application developers flexibility regarding the dependencies that they use, including using:

- libraries that aren't available in a distribution or runtime
- different versions of libraries from the ones that are in a distribution or runtime
- patched versions of libraries

Sandboxes
^^^^^^^^^

With Flatpak, each application is built and run in an isolated environment, which is called the 'sandbox'. Each sandbox contains an application and its runtime. By default, the application can only access the contents of its sandbox. Access to user files, network, graphics sockets, subsystems on the bus and devices have to be explicitly granted. Access to other things, such as other processes, is deliberately not possible.

By necessity, some resources that are inside the sandbox need to be exposed outside, to be used by the host system. These are known as 'exports', since they are files that are exported out of the sandbox, and include things like the application's ``.desktop`` file and icon.

Portals
^^^^^^^

Portals are a mechanism through which applications can interact with the host environment from within a sandbox. They give the ability to interact with data, files and services without the need to add sandbox permissions.

Examples of capabilities that can be accessed through portals include:

* Opening files with a native file chooser dialog
* Opening URIs
* Printing
* Showing notifications
* Taking screenshots
* Inhibiting the user session from ending, suspending, idling or getting switched away
* Getting network status information

Interface toolkits can implement transparent support for portals. If an application uses one of these toolkits, there is no additional work required to access them.

Applications that aren't using a toolkit with support for portals can refer to the `xdg-desktop-portal API documentation <https://flatpak.github.io/xdg-desktop-portal/portal-docs.html>`_ for information on how to access them.

Repositories
^^^^^^^^^^^^

Flatpak applications and runtimes are typically stored and published using repositories, which behave very similarly to Git repositories. A Flatpak repository can contain a single object or multiple objects, and each object is versioned, which allows upgrading and even downgrading.

Each system which is using Flatpak can be configured to access any number of remote repositories. Once a system has been configured to access a 'remote', the remote repository's content can be inspected and searched, and it can be used as the source of applications and runtimes.

When an update is performed, new versions of installed applications and runtimes are downloaded from the relevant remotes. Like with Git, only the difference between versions is downloaded, which makes the process very efficient.

Under the hood
--------------

Flatpak uses a number of pre-existing technologies. It generally isn't necessary to be familiar with these in order to use Flatpak, although in some cases it might be useful. They include:

* The `bubblewrap <https://github.com/projectatomic/bubblewrap>`_ utility from `Project Atomic <http://www.projectatomic.io/>`_, which lets unprivileged users set up and run containers, using kernel features such as:

  * Cgroups
  * Namespaces
  * Bind mounts
  * Seccomp rules

* `systemd <https://www.freedesktop.org/wiki/Software/systemd/>`_ to set up cgroups for sandboxes
* `D-Bus <https://www.freedesktop.org/wiki/Software/dbus/>`_, a well-established way to provide high-level APIs to applications
* The OCI format from the `Open Container Initiative <https://www.opencontainers.org/>`_, as a convenient transport format for single-file bundles
* The `OSTree <https://ostree.readthedocs.io/en/latest/>`_ system for versioning and distributing filesystem trees
* `Appstream <https://www.freedesktop.org/software/appstream/docs/>`_ metadata, to allow Flatpak applications to show up nicely in software-center applications

