Under the Hood
==============

This page provides an overview of how Flatpak works internally. While it
isn't necessary to be familiar with this in order to use Flatpak, some
people might find it interesting. Knowing about Flatpak's architecture also
helps to get a better understanding of how and why it works the way it does,
from a user and application developer perspective.

"Git for apps"
--------------

Flatpak is built on top of a technology called `OSTree
<https://ostreedev.github.io/ostree/introduction/>`_, which is
influenced by and very similar to the Git version control system. Like Git,
OSTree allows versioned data to be tracked and to be distributed between
different repositories. However, where Git is designed to track source files,
OSTree is designed to track binary files and other large data.

Internally, Flatpak therefore works in a similar way to Git, and many Flatpak
concepts are analogous to Git concepts. Like Git, Flatpak uses repositories
to store data, and it tracks the differences between versions.

With Flatpak, each application, runtime and extension is a branch in a
repository. An identifier triple, such as ``com.company.App/x86_64/stable``
is a reference to that branch. The output of a Flatpak build process is a
directory of files which is committed to one of these branches.

When an application is installed with Flatpak, it is pulled from the remote
into a new branch in a local repository. Links are then generated which point
from the right places in the filesystem to the application's files in the
repository (these are `hard links <https://en.wikipedia.org/wiki/Hard_link>`_,
which are fast to resolve and disk space efficient). In other words, every
application that is installed is stored in a local version control repository,
and is then mapped into the local filesystem.

Version tracking is therefore a core part of Flatpak's architecture, and
this makes updating software to the latest version very efficient. Versioning
also makes rollbacks possible, so it's easy to go back to a previous version,
should that be required.

Storing applications in a local OSTree repository has other advantages. For
example, it allows files that are stored on disk to be deduplicated, so
the same file that belongs to multiple applications (or runtimes) is only
stored once.

This `blog post <https://blogs.gnome.org/alexl/2017/10/02/on-application-sizes-and-bloat-in-flatpak/>`_
explains the underlying structure of a Flatpak repository in more detail.

Conditional permission system
-----------------------------

Since Flatpak 1.17.0, conditional permissions allow permissions to be
granted only when certain runtime conditions are satisfied, with
fallback to unconditional grants for compatibility with older versions.

Permissions are internally represented as:

- unconditionally allowed or denied
- a reset flag indicating whether the current layer overrides rules
  from lower layers
- a set of conditional rules under which the permission may be allowed

For example:

- ``--socket=NAME`` unconditionally allows the permission and resets any
  previously defined rules for that permission
- ``--nosocket=NAME`` unconditionally denies the permission and resets
  any previously defined rules
- ``--socket-if=NAME:CONDITION`` adds a conditional rule without
  resetting existing rules

Conditions may be negated using ``!``.

Multiple conditional rules can be specified for the same permission. In
this case, the permission is granted if any condition evaluates to true.

Duplicate conditions are ignored. The order of conditions does not
affect evaluation.

If no conditional rules are present, the permission is granted only if
it is unconditionally allowed.

If conditional rules are present, the permission is granted if any
condition evaluates to true, and denied otherwise, unless it is also
unconditionally allowed.

If an unconditional entry follows a conditional entry for the same
grant in commandline flags, the earlier unconditional entry is treated
as backwards compatibility fallback and does not affect the final
permission state. So the following is effectively treated as
``--socket-if=x11:!has-wayland`` in Flatpak versions supporting
conditional permissions::

  --socket=x11
  --socket-if=x11:!has-wayland

Permissions are written to metadata using the following rules:

- Unconditionally allowed permissions are written as ``NAME``
- Unconditionally denied permissions are written as ``!NAME``
- Conditionally allowed permissions are written as:

  - unconditional ``NAME`` entry for compat
  - ``if:NAME:CONDITION`` entries

If the permission resets previously defined rules, an explicit ``!NAME``
entry is written first, followed by the unconditional ``NAME`` entry and
then the ``if:NAME:CONDITION`` entries. This is omitted when saving an
application's own metadata, as opposed to overrides.

When parsing metadata, a non-negated unconditional ``NAME`` entry
appearing before a ``if:NAME:CONDITION`` entry is treated as a
compatibility fallback and does not affect the final permission
state. Eg. ``sockets=x11;if:x11:!has-wayland;`` is effectively treated
as ``if:x11:!has-wayland`` in Flatpak versions supporting
conditional permissions.

The ``fallback-x11`` socket, on pre-1.17 Flatpak versions implicitly
granted ``x11`` access and at runtime X11 access was suppressed when
Wayland was available, while on newer Flatpak (1.17+) it is internally
converted to the conditional syntax ``if:x11:!has-wayland``. When saving
the metadata, Flatpak converts ``if:x11:!has-wayland`` back to
``fallback-x11`` only when it is the sole conditional on ``x11``. If
additional conditionals are present, the new syntax is written directly
and older Flatpak versions will not understand the conditional entries.
A conditional grant for ``fallback-x11`` is not allowed.

Underlying technologies
-----------------------

Flatpak utilises a number of pre-existing technologies. These include:

* The `bubblewrap <https://github.com/containers/bubblewrap>`_ utility from
  `Project Atomic <https://projectatomic.io/>`_, which lets unprivileged
  users set up and run containers, using kernel features such as:

  * Namespaces
  * Bind mounts
  * Seccomp rules

* `systemd <https://www.freedesktop.org/wiki/Software/systemd/>`_ to set up
  cgroups for sandboxes
* `D-Bus <https://www.freedesktop.org/wiki/Software/dbus/>`_, a
  well-established way to provide high-level APIs to applications
* The `OSTree <https://ostreedev.github.io/ostree/>`__ system for
  versioning and distributing filesystem trees
* The OCI format from the `Open Container Initiative
  <https://opencontainers.org/>`_, as an alternative to OSTree used by the
  `Fedora infrastructure
  <https://blog.fishsoup.net/2018/12/04/flatpaks-in-fedora-now-live/>`__
* Flatpak can use either OSTree or OCI for single-file bundles.
* `Appstream <https://www.freedesktop.org/software/appstream/docs/>`_ metadata,
  to allow Flatpak applications to show up nicely in software center applications
