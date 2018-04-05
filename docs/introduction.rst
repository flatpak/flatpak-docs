Introduction to Flatpak
=======================

Flatpak is a framework for distributing desktop applications on Linux. It has been created by developers who have a long history of working on the Linux desktop, and is run as an independent open source project.

Target audience
---------------

Flatpak can be used by all kinds of desktop applications, and aims to be as agnostic as possible regarding how applications are built. There are no requirements regarding which programming languages, build tools or other technologies are used.

It can be used by applications that target multiple platforms, or by those which are Linux-specific. Applications can be open source or proprietary (although some distribution services, like `Flathub <https://flathub.org/>`_, have restrictions in this respect).

The only technical requirements made by Flatpak are that applications follow a small number of Freedesktop standards, in order to enable desktop integration.

Reasons to use Flatpak
----------------------

Flatpak has some major advantages over other approaches to distributing applications on Linux. First and foremost, Flatpak allows a single application build to be installed and run on virtually any Linux distribution. `Flathub <https://flathub.org/>`_ provides a centralized service for distributing Flatpaks on all distributions, which makes it possible for application developers to target the entire Linux desktop market from one place.

Flatpak also offers native integration for the main Linux desktops, so that users can easily browse, install, run and use Flatpak applications through their existing desktop environment and tools.

Other benefits for developers include:

- **Forward-compatibility**. This means that the same Flatpak will run on different versions of the same distribution, including versions that haven't been released yet. This doesn't require any changes or management by application developers.
- **Stable, maintained, platforms, called runtimes**. These contain collections of dependencies, which can be used by applications, and which can take a lot of the work out of application development.
- **Bundling**, which allows application developers to ship almost any dependency or library as part of their application. This gives complete control over which software is used to build applications.
- **Consistent runtime environments**, which are the same across devices. This ensures that applications perform as intended. It also makes it easier to identify bugs and to do testing.

Finally, while Flatpak does provide a centralized service for distributing applications, it also allows decentralized hosting and distribution, so that application developers or downstreams can host their own applications and application repositories.
