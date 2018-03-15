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

How it works
------------

Flatpak can be understood through a small number of key concepts. These also help to explain how it differs from traditional software packages.

.. image:: ../images/diagram.svg

Runtimes
^^^^^^^^

Runtimes provide the basic dependencies that are used by applications. Each application must be built against a runtime, and this runtime must be installed on a host system in order for the application to run (runtimes are usually automatically installed as required). Multiple different runtimes can be installed at the same time, including different versions of the same runtime.

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

The flatpak command
--------------------

``flatpak`` is the primary Flatpak command, and can be used to find, install and remove applications. Specific commands are appended to ``flatpak``. For example, the command to install something is ``flatpak install``.

``flatpak --help`` provides the full list of available commands, as does the :doc:`flatpak-command-reference`.

Identifiers
-----------

Flatpak identifies each application and runtime using a unique identifier, which takes the form of an inverse DNS address, such as ``com.company.App``. The final segment of this address is the object's name, and the preceding part is the domain that it belongs to.

.. note::

  Flatpak's reverse DNS identifiers prevent naming conflicts. They also correspond to the identifiers used by D-Bus.

In order to ensure uniqueness, the domain part of each identifier should correspond to a registered DNS address. This means using a domain from a website, either for an application or an organization. For instance, the developers of ``com.company.App`` would be expected to have registered the ``company.com`` web domain.  Multiple applications can belong to the same domain, such as ``com.company.App1`` and ``com.company.App2``.

If you do not have a registered domain for your application, it is easy to use a third party website to get one. For example, Github allows the creation of personal pages that can be used for this purpose. Here, a personal namespace of ``name.github.io`` could be used as the basis of application identifier ``io.github.name.App``.

If an application provides a D-Bus service, the D-Bus service name is expected to be the same as the application name.

Identifier triples
^^^^^^^^^^^^^^^^^^

Typically it is sufficient to refer to objects using their reverse DNS identifier. However, in some situations it is necessary to refer to a specific version of an object, or to a specific architecture. For example, some applications might be available as a stable and a testing version, in which case it is necessary to specify which one you want to install.

Flatpak allows architectures and versions to be specified using an object's identifier triple. This takes the form of ``name/architecture/branch``, such as ``com.company.App/i386/stable``. (Branch is the term used to refer to versions of the same object.) The first part of the triple is the reverse DNS name, the second part is the architecture, and the third part is the branch.

The Flatpak CLI will provide feedback if an identifier triple is required, instead of the standard object ID.

System versus user
------------------

Flatpak commands can be run either system-wide or per-user. Applications and runtimes that are installed system-wide are available to and shared between all users on the system. Applications and runtimes that are installed per-user are only available to the user that installed them.

The same principle applies to repositories - repositories that have been added system-wide are available to all users, whereas per-user repositories can only be used by a particular user.

Flatpak commands are run system-wide by default, since it reduces redundancy. If you are installing applications for day-to-day usage, it is recommended to stick with this default behavior.

However, running commands per-user can be useful for testing and development purposes, since objects that are installed in this way won't be available to other users on the system. To do this, use the ``--user`` option, which can be used in combination with most ``flatpak`` commands.

Commands behave in exactly the same way if they are run per-user rather than system-wide.

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

