Introduction to Flatpak
=======================

Flatpak is a framework for distributing desktop applications across various
Linux distributions. It was created by developers with a long history of
working on the Linux desktop and is run as an independent open-source project.

Terminology
-----------

- Flatpak: a system for building, distributing, and running sandboxed desktop
  applications on Linux.
- Flatpak application: an application installed via the ``flatpak`` command or through a
  graphical interface, such as GNOME Software or KDE Discover.
- Runtime: also called platform, an integrated environment providing basic
  utilities needed for a Flatpak application to work.
- BaseApp: an integrated set of additional libraries and frameworks, such as Electron, for Flatpak
  applications that need more than just the basic runtime.
- Flatpak bundle: a single-file export format containing a Flatpak appplication or
  runtime.

Target audience
---------------

Flatpak can be used by all kinds of desktop applications and aims to be as
agnostic as possible in terms of how applications are built. There are no
requirements regarding the programming languages, build tools, toolkits, or
frameworks used.

While Flatpak only runs on Linux, it can be used by applications that target
other operating systems as well as those that are Linux-specific. Applications
can be open-source or proprietary (although some distribution services, like
`Flathub <https://flathub.org/>`_, can have restrictions in this respect).

Flatpak's only technical requirements are that applications follow a few
Freedesktop standards to enable desktop integration (see :doc:`conventions`).

Issues with current model of packaging
--------------------------------------

Understanding the problems with the current model
of packaging applications helps explain why Flatpak exists:

- **Duplicated work packaging apps**: many Linux distributions have their own
  package manager, package format, and package repository. This leads to a lot of
  duplicated work, with different maintainers packaging the same application
  for various distributions. Otherwise, developers either need to learn the format of
  each distribution or ignore most distributions and support only a few. This makes
  the Linux desktop a difficult platform for software vendors to target.
- **Limited to apps that are packaged**: not every application is natively
  available in every Linux distribution. If an application is not available in
  a specific distribution, users must manually download the archive of
  the application, then extract it and hope the application will launch.
- **Limited to distributions that have the apps**: users are limited to
  distributions that have the necessary applications for their workflow.
  This reduces the number of suitable distributions for a user.
- **Hard to innovate in OS space**: distribution maintainers spend much of their time
  packaging applications rather than focus on their end goals. This delays each
  distribution's progress.
- **Old and outdated packages**: LTS distributions often have very old
  versions of applications packaged natively. This complicates bug
  reproducibility since applications run in different environments, and
  developers often have little control over how their applications are packaged
  by distributions.

Flatpak addresses these issues by enabling developers to distribute
applications from one source and target the entire Linux desktop.

Reasons to use Flatpak
----------------------

Flatpak offers major advantages over most system package managers:

- **Universality**: Flatpak allows applications to be installed and run on virtually any Linux
  distribution, including non-GNU distributions, systemd-free distributions,
  distributions with a read-only operating system (OS), and various architectures without requiring
  the developer to have access to the relevant hardware.
- **Space for innovation**: Flatpak allows distribution maintainers to focus on
  innovating their distribution without being burdened by packaging concerns.
- **Stability**: breakages in Flatpak applications won't affect the system,
  since they run in isolated environments.
- **Rootless install**: Flatpak does not require elevated privileges to install
  applications or runtimes.
- **Sandboxed applications**: Flatpak increases security, one of its main goals, by
  isolating applications from one another and limiting their access to the host environment.

Flatpak also offers advantages over other universal approaches to Linux application distribution:

- **Decentralized by design**: while Flatpak does provide a centralized service for distributing
  applications, it also offers decentralized hosting and distribution, allowing developers or
  downstreams to host their own applications and application repositories.
- **Desktop integration**: Flatpak offers native integration with the main Linux desktops,
  allowing users to easily browse, install, run, and manage Flatpak
  applications through their existing desktop environment and tools.
- **Space efficiency**: Flatpak saves storage by deduplicating libraries and
  other files used by multiple applications.
- **Delta updates**: only changed files are downloaded during updates.

Other benefits for developers include:

- **Forward-compatibility**: the same Flatpak application can run on different versions of
  the same distribution, including unreleased versions, without changes from the application
  developer.
- **Bundling**: developers can ship almost any dependency or library as part of
  their applications, giving them control over the softwares used to build them.
- **Consistent application environments**: because application environments are identical across
devices, applications run as expected; bug identification, as well as testing, is also made easier.
- **Branches**: developers can distribute applications from different branches
  (e.g. ``stable``, ``beta``, etc.) while the application name can stay the same.
- **Maintained platforms**: Flatpak's maintained runtimes contain collections
  of dependencies for applications to use, easing development.

In general, Flatpak is best suited for desktop applications. While command-line
applications also work, Flatpak may not be suitable in some cases:

- The application needs to elevate privileges using ``su``, ``sudo``, ``pkexec``, etc.
  Flatpak cannot run SUID binaries inside the sandbox.
- The application requires access to ``/proc`` on the host or unfiltered
  access to processes. This is not allowed as Flatpak has a private ``proc``.
- The application uses a syscall blocklisted by Flatpak's seccomp filter. For
  example, Flatpak won't allow spawning sub-namespaces in the sandbox.
- Kernel modules or drivers are non-application packages and won't work
  inside a flatpak.

In general, if the sandbox prohibits an application's core functionality or becomes
too inconvenient or obtrusive, Flatpak may not be the most suitable packaging choice.

Flatpak also won't export udev rules or systemd services from the sandbox
to the host, requiring manual configuration after installing the Flatpak package.

Information about Flatpak's internals can be found in :doc:`under-the-hood`.