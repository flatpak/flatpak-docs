End User Documentation
======================

This documentation is directed to technical users configuring their own system. It it
likely unnecessary if you're using an integrated desktop environment like GNOME.

Portals
-------

Desktop portals are a framework for securely sharing resources between sandboxed
applications and the host system.

The ``xdg-desktop-portal`` service will automatically be started via D-Bus activation
when necessary. The ``XDG_CURRENT_DESKTOP`` environment variable defines what desktop
you're using, and ``xdg-desktop-portal`` will use this to determine which portals need
to be started.

You'll only need to install the correct portals, and D-Bus activation will handle
stating them at the right time. Having a working D-Bus setup if a hard requirement for
using Flatpak.

GTK applications
----------------

GTK applications rely on ``xdg-desktop-portal-gtk`` running. This portal exposes things
like ``gsettings``, which are required for GTK applications to render fonts properly.
Without this portal, you'll notice very poorly rendered fonts on GTK applications.

While this portal is described as a GNOME-specific portal, in reality, it is needed on
any desktop if you intend to run GTK applications.

In order to use multiple portals, you should run something like:

.. code::

    export XDG_CURRENT_DESKTOP=whatever:gnome

For example, for ``sway``, you'll need to run:

.. code::

    export XDG_CURRENT_DESKTOP=sway:gnome
