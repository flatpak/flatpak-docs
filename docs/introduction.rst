Introduction to Flatpak
=======================

Flatpak is a framework for distributing desktop applications across various
Linux distributions. It was created by developers with a long history of
working on the Linux desktop and is run as an independent open source project.

Terminology
-----------

- Flatpak: a system for building, distributing, and running sandboxed desktop applications on Linux.
- Flatpak application: an application installed via the ``flatpak`` command or through a
  graphical interface, such as GNOME Software or KDE Discover.
- Runtime: also called platform, an integrated environment providing basic
  utilities needed for a Flatpak application to work.
- BaseApp: an integrated set of additional libraries and frameworks, such as Electron, for Flatpak
  applications that need more than just the basic runtime.
- Flatpak bundle: a single-file export format containing a Flatpak application or runtime.

Target audience
---------------

Flatpak can be used by all kinds of desktop applications and aims to be as
agnostic as possible in terms of how applications are built. There are no
requirements regarding which programming languages, build tools, toolkits
or frameworks can be used.

While Flatpak only runs on Linux, it can be used by applications that target
other operating systems as well as those that are Linux-specific. Applications
can be open source or proprietary (although some distribution services, like
`Flathub <https://flathub.org/>`_, can have restrictions in this respect).

Flatpak's only technical requirements are that applications follow a
small number of Freedesktop standards to enable desktop integration
(see :doc:`conventions`).

Issues with the current packaging model
---------------------------------------

It is important to understand the problems with the current model
of packaging applications to understand the existence of Flatpak:

- **Fragmentation and duplicated work**: each Linux distribution has their own package
  managers and package formats that are incompatible with each other.
  This leads to a lot of fragmentation and duplicated work since the same application
  needs to be packaged multiple times for various distributions. Furthermore, developers
  either need to learn the format of every distribution or support only a few. This makes
  the Linux desktop a difficult platform for software vendors to target.
- **Limited to apps that are packaged**: not every application is packaged for
  every Linux distribution. If an application isn't packaged for a specific
  distribution, users are left with limited and unreliable options, often technical in nature.
- **Limited to distributions that have the apps**: users are limited to only the
  distributions that have all the necessary applications for their workflow.
- **Hard to innovate in OS space**: distribution maintainers spend a huge amount of
  time and effort in packaging rather than focusing on improving their distributions.
- **Old and outdated packages**: LTS distributions often have very old versions of applications
  packaged. This complicates bug reproducibility or leads to support
  requests for bugs that have been fixed upstream in newer versions.

Flatpak addresses these issues by enabling developers to distribute
applications from one source and target the entire Linux desktop.

Reasons to use Flatpak
----------------------

Flatpak offers major advantages over most system package managers:

- **Universality**: Flatpak allows applications to be installed and run on virtually any Linux
  distribution, including non-GNU distributions, systemd-free distributions,
  immutable distributions and various architectures without requiring the
  developer to have access to the relevant hardware.
- **Space for innovation**: Flatpak allows distribution maintainers to focus on
  innovating their distribution without being burdened by packaging concerns.
- **Stability**: breakages in Flatpak applications do not affect the system
  since they run in isolated environments.
- **Rootless install**: Flatpak does not require elevated privileges to install
  applications or runtimes.
- **Sandboxed applications**: one of Flatpak's main goals is to increase the security of desktop
  systems. This is achieved by isolating applications from one another and limiting their
  access to the host environment.

Flatpak also offers advantages over other universal approaches to Linux application distribution:

- **Decentralized by design**: Flatpak offers decentralized hosting and distribution,
  allowing developers or downstreams to host their own applications and
  application repositories.
- **Desktop integration**: Flatpak offers native integration with many Linux desktop environments, allowing
  users to easily browse, install, run, and manage Flatpak applications.
- **Space efficiency**: Flatpak saves storage by deduplicating libraries and other files used by multiple
  applications.
- **Delta updates**: only updated data is downloaded during updates, saving network bandwidth.

Other benefits for developers include:

- **Forward-compatibility**: the same Flatpak application can run on different versions
  of a distribution, including unreleased versions, without needing any changes.
- **Bundling**: this allows application developers to ship almost any
  dependency or library as part of their applications, giving them complete
  control over their applications.
- **Consistent application environments**: Flatpak provides a consistent and identical
  application runtime environment across devices and distributions. This makes
  bug identification and testing easier.
- **Branches**: this allows application developers to distribute multiple branches of
  an application (e.g. ``stable``, ``beta``, etc.) while retaining the same name.
- **Maintained platforms**: Flatpak runtimes contain a collection
  of common dependencies for the applications to use, easing
  application development and maintenance.


In general, Flatpak is best suited for desktop applications.
While command-line applications also work, Flatpak may not be suitable in some
cases:

- The application needs to elevate privileges using ``su``, ``sudo``, ``pkexec``,
  etc. Flatpak cannot run SUID binaries inside the sandbox.
- The application requires access to ``/proc`` on the host or unfiltered access
  to processes. This is not allowed as Flatpak has a private ``proc``.
- The application uses a syscall blocklisted by Flatpak's seccomp filter.
  For example, Flatpak won't allow spawning sub-namespaces
  in the sandbox.
- Kernel modules or drivers are non-application packages and won't work
  as a Flatpak.

In general, if the sandbox prohibits an application's core functionality
or becomes too inconvenient or obtrusive, Flatpak may not be
the most suitable packaging choice.

Flatpak also won't export udev rules or systemd services from the sandbox
to the host and requires manual configuration after installing the
Flatpak package.

Information about Flatpak's internals can be found in :doc:`under-the-hood`.
