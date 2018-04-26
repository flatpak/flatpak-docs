Debugging tools in the SDK
==========================

The normal runtimes that applications use when running inside a Flatpak sandbox are
optimized for size and don't include debugging tools. Every runtime has a matching
SDK, which includes the same runtime components (libraries, etc), but additionally
has the necessary bits and pieces to build and debug applications.

The freedesktop SDK (on which many others are based), includes tools such
as gdb, strace, nm, dbus-send, dconf and many others.
