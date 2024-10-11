Basic concepts
==============

Flatpak can be understood through a few key concepts. Familiarizing yourself
with these will be useful for learning to use Flatpak
from the command line or building applications with it.

.. image:: ../images/diagram.svg

Runtimes
--------

Runtimes provide the basic dependencies used by applications. Each
application must be built against a runtime, and this runtime must be
installed on the host system for the application to run. (Flatpak
can automatically install the required runtime for an application.) Multiple
runtimes and different versions of the same runtime can be installed
alongside each other.

Runtimes are distribution agnostic and do not depend on a particular distribution
version. This means that they provide a stable, cross-distribution base
for applications and allow applications to work irrespective
of operating system updates.

Bundled libraries
-----------------

If an application requires dependencies that aren't in its runtime, they
can be bundled with the application. This gives application developers
flexibility in their choice of dependencies, allowing them to use:

- libraries that aren't available in a runtime
- different versions of libraries from those available in a runtime
- patched versions of libraries

Sandboxes
---------

With Flatpak, each application is built and run in an isolated environment
called the 'sandbox'. Each sandbox contains the application and
its runtime. By default, the application can only access the contents of
its sandbox. Access to user files, network, graphics sockets, subsystems on
the bus, and devices have to be explicitly granted. Access to other resources,
such as other processes, is deliberately not possible.

By necessity, some resources inside the sandbox need to be exported
outside to be used by the host system. These are known as 'exports'
and include resources such as the application's desktop file and
its icon.

Portals
-------

Portals are a mechanism through which applications can interact with the
host environment from within the sandbox. They enable access
to data, files and services without requiring additional static sandbox permissions.

Examples of capabilities that can be accessed through portals include opening
files through a file chooser dialog or printing. Interface toolkits can
offer transparent support for portals, ensuring secure and out-of-the-box
access to resources outside the sandbox.

More information about portals can be found in :doc:`sandbox-permissions`.

Repositories
------------

Flatpak applications and runtimes are typically stored and published using
repositories, which behave very similarly to Git repositories: a Flatpak
repository can contain a single object or multiple objects, and each object
is versioned, allowing for upgrades and even downgrades.

Each system using Flatpak can be configured to access any number of
remote repositories. Once a system has been configured to access a 'remote',
the remote repository's content can be inspected, searched, and
used as a source of applications and runtimes.

When an update is performed, new versions of installed applications and
runtimes are downloaded from the relevant remotes. Like Git, only
the parts that have changed between versions are downloaded, making the process
very efficient.
