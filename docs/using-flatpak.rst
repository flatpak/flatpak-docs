Using Flatpak
=============

This page provides an introduction to the ``flatpak`` command line interface,
and explains essential technical conventions as well as the most common
commands.

End users shouldn't generally need to use this page or the Flatpak command
line interface, since Flatpak can be easily used through graphical
management tools, such as GNOME Software and Flatseal, though they are of course 
free to use the command line if they prefer!


The flatpak command
--------------------

``flatpak`` is the primary Flatpak command, to which specific commands are
appended. For example, the command to install something is ``flatpak install``
and the command to uninstall is ``flatpak uninstall``. Additional subcommands 
can be listed with ``flatpak --help``.

Identifiers
-----------

Flatpak identifies each project (object) using a two-part identifier (ID),
in reverse-DNS style. The final part of this identifier is the application's name.
The preceding segments identify the developer so that the same developer can have 
multiple applications, like ``com.company.App1`` and ``com.company.App2``. 
Some examples are:: 

``org.gnome.Epiphany`` (The first two segments are the organization, the final segment is the application)
``io.github.username.Application`` (the organization is the first three segments)


Identifier triples
``````````````````

Typically it is sufficient to refer to objects using their ID. However,
in some situations it is necessary to refer to a specific version of an
object, or to a specific architecture. For example, some applications might
be available as a stable and a testing version, in which case it is necessary
to specify which one you want to install.

Flatpak allows architectures and versions to be specified using an object's
identifier triple. This takes the form of ``name/architecture/branch``,
such as ``com.company.App/i386/stable``. (Branch is the term used to refer
to versions of the same object.) The first part of the triple is the ID,
the second part is the architecture, and the third part is the branch.

Identifier triples can also be used to specify just the architecture
or the branch, by leaving part of the triple blank. For example,
``com.company.App//stable`` would just specify the branch, and
``com.company.App/i386//`` just specifies the architecture.

The Flatpak CLI provides feedback if the branch of an object needs to be 
specified - it will list all branches of an object if you just specify the 
ID, on your current architecture. GUI Implementations will typically 
include a menu for you to select which branch to install of an app.

System versus user
------------------

Flatpak commands can be run either system-wide or per-user. Objects that 
are installed system-wide are available to all users on the system. 
Objects that are installed per-user are only available to the users that 
installed them.

The same principle applies to repositories - repositories that have been
added system-wide are available to all users, whereas per-user repositories
can only be used by a particular user.

Flatpak commands are run system-wide by default. If you are installing 
applications for day-to-day usage, it is recommended to use ``--user`` if 
you don't need your apps available for all users on your system. Installing 
for the system can be specified with the ``--system`` argument.

Running commands per-user can also be useful for testing and development
purposes, since objects that are installed in this way won't be available
to other users on the system. To do this, use the ``--user`` option, which
can be used in combination with most ``flatpak`` commands.

Commands behave in exactly the same way if they are run per-user rather
than system-wide, but running them system-wide uses polkit to authenticate the user.
Flatpak apps install to `$XDG_DATA_HOME/flatpak`. This is typically `$HOME/.local/share/flatpak` 
for user-installed apps, and `/var/lib/flatpak` for system-wide installations.
Per-user app configurations are available under `$HOME/.var`. This may or may not be 
followed by an app, depending on whether or not it follows the appropriate environment 
variables correctly.

Basic commands
--------------

This section covers basic commands needed to install, run and manage Flatpak
applications. For the full list of Flatpak commands, run ``flatpak --help``
or see the :doc:`flatpak-command-reference`.

List remotes
````````````

To list the remotes that you have configured on your system, run::

  $ flatpak remotes

This gives a list of the existing remotes that have been added. The list
indicates whether each remote has been added per-user or system-wide.
Additional remote-related commands are available, and are listed with ``flatpak --help``.

Add a remote
````````````

The most convenient way to add a remote is by using a ``.flatpakrepo`` file,
which includes both the details of the remote and its GPG key::

 $ flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo

Here, ``flathub`` is the local name that is given to the remote. The URL
points to the remote's ``.flatpakrepo`` file. ``--if-not-exists`` stops the
command from producing an error if the remote already exists.

Remove a remote
```````````````

To remove a remote, run::

 $ flatpak remote-delete flathub

In this case, ``flathub`` is the remote's local name.

Search
``````

Applications can be found in any of your remotes using the ``search``
command. For example::

 $ flatpak search krita

Search will return any applications matching the search terms. Each search
result includes the application ID and the remote that the application is
in. In this example, the search term is ``krita``.

Install applications
````````````````````

To install an application, run::

 $ flatpak install flathub org.kde.krita

Here, ``flathub`` is the name of the remote the application is to be installed
from, and ``org.mozilla.firefox`` is the ID of the application. You can specify a 
part of an ID, and you will be suggested apps that contain that part of the 
ID, rather than needing to know the full ID.

Sometimes, an application will require a particular runtime, and this will
be installed prior to the application.

The details of the application to be installed can also be provided by a
``.flatpakref`` file, which can be either remote or local. To specify a
``.flatpakref`` instead of manually providing the remote and application
ID, run::

 $ flatpak install https://flathub.org/repo/appstream/org.kde.krita.flatpakref

If the ``.flatpakref`` file specifies that the application is to be installed
from a remote that hasn't already been added, you will be asked whether to
add it before the application is installed.

Since Flatpak 1.2, the ``install`` command can search for applications. A
simple::

 $ flatpak install krita

will confirm the remote and application exist, and proceed to install.

Running applications
````````````````````

Once an application has been installed, it can be launched using the ``run``
command and its application ID::

 $ flatpak run your.application.ID

Updating
````````

To update all your installed applications and runtimes to the latest version,
run::

 $ flatpak update

List installed applications
```````````````````````````

To list the applications and runtimes you have installed, run::

 $ flatpak list

Alternatively, to just list installed applications, run::

 $ flatpak list --app

There are more options available, which can be seen with ``flatpak list --help``.

Remove an application
`````````````````````

To remove an application, run::

 $ flatpak uninstall your.application.ID

This will search your installed applications for the app you specified, 
and start the uninstall process. You can use a partial app ID in place of a full ID.

Troubleshooting
```````````````

Flatpak has a few commands that can help you to get things working again when
something goes wrong.

To remove runtimes and extensions that are not used by installed applications,
use::

 $ flatpak uninstall --unused

To fix inconsistencies with your local installation, use::

 $ flatpak repair

This command is ran on the root installation by default, 
append ``--user`` to repair the user installation.

Flatpak also has a number of commands to manage the portal permissions of
installed apps. To reset all portal permissions for an app, use ``flatpak
permission-reset``::

 $ flatpak permission-reset your.application.ID

There are more available flatpak permission commands, such as 
``permission-remove``, ``permission-set``, and ``permission-show``.
These can be managed with a GUI tool such as Flatseal(insert link to Flatseal here)
..Maybe ``override`` can be included?
File access can be managed with the corresponding `flatpak document*` command.

To find out what changes have been made to your Flatpak installation over time,
you can take a look at the logs (since 1.2)::

 $ flatpak history

You can list running Flatpak applications with the following command::

 $ flatpak ps 

 This will list all currently running flatpaks, their runtime, and their 
 PIDs (process IDs). You can terminate a running flatpak with::

 $ flatpak kill the.application.ID

The full application ID is required, a portion of one will not work.

Additional commands available
`````````````````````````````
Flatpak can pin a runtime to prevent automatic removal, like so::

 $ flatpak pin org.freedesktop.Platform


It is posssible to enter a running Flatpak sandbox, and manipulate it as 
if it was an immutable operating system. To do this, run::

  $ flatpak enter your.running.Application command

The command can be a shell, or any other command of your choosing.

.. A specific version of an app can be set as the default, with the `make-current` command:
 This needs someone else to document it as well.

.. Someone please document the create-usb command.
 Applications or runtimes can be transferred to removable media, with the ``create-usb`` command::


..The "Build applications" section from ``--help`` needs to be documented as well.
