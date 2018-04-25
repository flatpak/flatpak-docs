Introduction to Flatpak
=======================

Flatpak is a framework for distributing desktop applications on Linux. It has been created by developers who have a long history of working on the Linux desktop, and is run as an independent open source project.

Target audience
---------------

Flatpak can be used by all kinds of desktop applications, and aims to be as agnostic as possible regarding how applications are built. There are no requirements regarding which programming languages, build tools, toolkits or frameworks can be used.

While Flatpak only runs on Linux, it can be used by applications that target other operating systems, as well as those that are Linux-specific. Applications can be open source or proprietary (although some distribution services, like `Flathub <https://flathub.org/>`_, can have restrictions in this respect).

The only technical requirements made by Flatpak are that applications follow a small number of Freedesktop standards, in order to enable desktop integration (see :doc:`conventions`).

Reasons to use Flatpak
----------------------

Flatpak has some major advantages over other approaches to distributing applications on Linux. First and foremost, Flatpak allows a single application build to be installed and run on virtually any Linux distribution. It can also be used in combination with `Flathub <https://flathub.org/>`_, a centralized service for distributing applications on all distributions. This makes it possible for application developers to target the entire Linux desktop market from one place.

Flatpak also offers native integration for the main Linux desktops, so that users can easily browse, install, run and use Flatpak applications through their existing desktop environment and tools.

Other benefits for developers include:

- **Forward-compatibility:** the same Flatpak can be run on different versions of the same distribution, including versions that haven't been released yet. This doesn't require any changes or management by application developers.
- **Maintained platforms:** called runtimes, these contain collections of dependencies, which can be used by applications, and which can take a lot of the work out of application development.
- **Bundling:** this allows application developers to ship almost any dependency or library as part of their application. This gives complete control over which software is used to build applications.
- **Consistent application environments:** because these are are the same across devices, applications perform as intended. This also makes it easier to identify bugs and to do testing.

Finally, while Flatpak does provide a centralized service for distributing applications, it also allows decentralized hosting and distribution, so that application developers or downstreams can host their own applications and application repositories.

Information about Flatpak's internals can be found in :doc:`under-the-hood`.
