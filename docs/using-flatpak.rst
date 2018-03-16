Using Flatpak
=============

This page provides an introduction to the ``flatpak`` command line tool, including the most common commands needed to use Flatpak.

The flatpak command
--------------------

``flatpak`` is the primary Flatpak command, to which specific commands are appended. For example, the command to install something is ``flatpak install`` and the command to uninstall is ``flatpak uninstall``.

Identifiers
-----------

Flatpak identifies each application and runtime using a unique identifier, which takes the form of an inverse DNS address, such as ``com.company.App``. The final segment of this address is the object's name, and the preceding part is the domain that it belongs to, which should belong to the developer. Multiple applications can belong to the same domain, such as ``com.company.App1`` and ``com.company.App2``.

Flatpak's reverse DNS identifiers prevent naming conflicts. They also correspond to the identifiers used by D-Bus.

.. note::

  Developers ought to ensure that the domain part of their application ID corresponds to a registered DNS address. This means using a domain from a website, either for an application or an organization. For instance, the developers of ``com.company.App`` would be expected to have registered the ``company.com`` web domain.

  If you do not have a registered domain for your application, it is easy to use a third party website to get one. For example, Github allows the creation of personal pages that can be used for this purpose. Here, a personal namespace of ``name.github.io`` could be used as the basis of application identifier ``io.github.name.App``.

  If an application provides a D-Bus service, the D-Bus service name is expected to be the same as the application name.

Identifier triples
``````````````````

Typically it is sufficient to refer to objects using their reverse DNS identifier. However, in some situations it is necessary to refer to a specific version of an object, or to a specific architecture. For example, some applications might be available as a stable and a testing version, in which case it is necessary to specify which one you want to install.

Flatpak allows architectures and versions to be specified using an object's identifier triple. This takes the form of ``name/architecture/branch``, such as ``com.company.App/i386/stable``. (Branch is the term used to refer to versions of the same object.) The first part of the triple is the reverse DNS name, the second part is the architecture, and the third part is the branch.

The Flatpak CLI provides feedback if an identifier triple is required, instead of the standard object ID.

System versus user
------------------

Flatpak commands can be run either system-wide or per-user. Applications and runtimes that are installed system-wide are available to all users on the system. Applications and runtimes that are installed per-user are only available to the users that installed them.

The same principle applies to repositories - repositories that have been added system-wide are available to all users, whereas per-user repositories can only be used by a particular user.

Flatpak commands are run system-wide by default. If you are installing applications for day-to-day usage, it is recommended to stick with this default behavior.

However, running commands per-user can be useful for testing and development purposes, since objects that are installed in this way won't be available to other users on the system. To do this, use the ``--user`` option, which can be used in combination with most ``flatpak`` commands.

Commands behave in exactly the same way if they are run per-user rather than system-wide.

Basic commands
--------------

This section covers basic commands needed to install, run and manage Flatpak applications. For the full list of Flatpak commands, run ``flatpak --help`` or see the :doc:`flatpak-command-reference`.

List remotes
````````````

To list the remotes that you have configured on your system, run::

  $ flatpak remotes

This gives a list of the existing remotes that have been added. The list indicates whether each remote has been added per-user or system-wide.

Add a remote
````````````

The most convenient way to add a remote is by using a ``.flatpakrepo`` file, which includes both the details of the remote and its GPG key::

 $ flatpak remote-add --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo

Here, ``flathub`` is the local name that is given to the remote. The URL points to the remote's ``.flatpakrepo`` file. ``--if-not-exists`` stops the command from producing an error if the remote already exists.

Remove a remote
```````````````

To remove a remote, run::

 $ flatpak remote-delete flathub

In this case, ``flathub`` is the remote's local name.

Search
``````

Applications can be found in any of your remotes using the ``search`` command. For example::

 $ flatpak search gimp

Search will return any applications matching the search terms. Each search result includes the application ID and the remote that the application is in. In this example, the search term is ``gimp``.

.. note::
  Prior to Flatpak 0.11.1, it was necessary to manually update the metadata for your remotes before search will work. This can be done by either running ``flatpak update`` or ``flatpak update --appstream``.

Install applications
````````````````````

To install an application, run::

 $ flatpak install flathub org.gimp.GIMP

Here, ``flathub`` is the name of the remote the application is to be installed from, and ``org.gimp.GIMP`` is the ID of the application.

Sometimes, an application will require a particular runtime, and this will be installed prior to the application.

The details of the application to be installed can also be provided by a ``.flatpakref`` file, which can be either remote or local. To specify a ``.flatpakref`` instead of manually providing the remote and application ID, run::

 $ flatpak install https://flathub.org/repo/appstream/org.gimp.GIMP.flatpakref

If the ``.flatpakref`` file specifies that the application is to be installed from a remote that hasn't already been added, you will be asked whether to add it before the application is installed.

Running applications
````````````````````

Once an application has been installed, it can be launched using the ``run`` command and its application ID::

 $ flatpak run org.gimp.GIMP

Updating
````````

To update all your installed applications and runtimes to the latest version, run::

 $ flatpak update

List installed applications
```````````````````````````

To list the applications and runtimes you have installed, run::

 $ flatpak list

Alternatively, to just list installed applications, run::

 $ flatpak list --app

Remove an application
`````````````````````

To remove an application, run::

 $ flatpak uninstall org.gimp.GIMP
