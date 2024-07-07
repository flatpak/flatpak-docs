Deploying configuration
=======================

This chapter is for system administrators who deploy flatpak applications and
their configuration data.

For automated deployment of flatpak apps, you can call `flatpak
install` as you normally would.

To deploy an application's configuration, follow the instructions in this chapter.

Configuration locations
-----------------------

Normally, applications follow the `XDG Base Directory Specification
<https://specifications.freedesktop.org/basedir-spec/basedir-spec-latest.html>`
and are expected to place files in the paths passed to them in the environment
variables ``$XDG_DATA_HOME``, ``$XDG_CONFIG_HOME``, ``$XDG_STATE_HOME``,
``$XDG_CACHE_HOME``.

Outside of flatpak, the default values for those variables are as follows:

 - ``XDG_DATA_HOME   = ~/.local/share`` - user-specific data files.
 - ``XDG_CONFIG_HOME = ~/.config`` - user-specific configuration files.
 - ``XDG_STATE_HOME  = ~/.local/state`` - user-specific data files.
 - ``XDG_CACHE_HOME  = ~/.cache`` - user-specific, non-essential cached data.

While running as a flatpak, however, the variables are set as follows.  If the
app is called ``org.example.AppName``, then the variables will be:

 - ``XDG_DATA_HOME   = ~/.var/app/org.example.AppName/data``
 - ``XDG_CONFIG_HOME = ~/.var/app.org.example.AppName/config``
 - ``XDG_STATE_HOME  = ~/.var/app/org.example.AppName/.local/state``
 - ``XDG_CACHE_HOME  = ~/.var/app/org.example.AppName/cache``

To maintain files in a configuration management system so that they can be
deployed later, you should pay special attention to the ``XDG_CONFIG_HOME``
locations for applications.

Applications that save user-created data may do so in ``XDG_DATA_HOME``; this
should be part of your backups.

Note that not all applications have a clean separation between configuration and
state.  You may want to see what each application puts in ``XDG_CONFIG_HOME``
versus ``XDG_STATE_HOME`` to decide what to persist.

Overrides
---------

Users may run ``flatpak override`` or a tool like ``com.github.tchx84.Flatseal``
to add or remove permissions from an application.  These modifications are
called **overrides**.

 - **App-specific overrides** go in a file called
   ``.local/share/flatpak/overrides/org.example.AppName``

 - **Global overrides** apply to all applications and go in a file called
   ``.local/share/flatpak/overrides/global``

Both kinds of overrides can be persisted in a configuration management system.
