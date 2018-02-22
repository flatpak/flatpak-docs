Using Flatpak
=============

This page provides an introduction to the most common commands needed to use Flatpak. It is not intended to be exhaustive or to cover all the options for each command (a full list of all the commands and their options can be found in `the command reference <flatpak-command-reference.html>`_).

.. note::
  Flatpak commands can be run either per-user or system-wide. All the examples in this guide use the default system-wide behavior.

Remotes
-------

Remotes are the repositories from which applications and runtimes can be installed.

List remotes
````````````

To list the remotes that you have configured on your system, run::

  $ flatpak remotes

This gives a list of the existing remotes that have been added. The list indicates whether each remote has been added per-user or system-wide.

Add a remote
````````````

Adding a remote allows you to search and list its contents, and to install applications from it. The most convenient way to add a remote is by using a ``.flatpakrepo`` file, which includes both the details of the remote and its GPG key::

 $ flatpak remote-add --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo

Here, ``flathub`` is the local name that is given to the remote. The URL points to the remote's ``.flatpakrepo`` file. ``--if-not-exists`` stops the command from producing an error if the remote already exists.

Remove a remote
```````````````

To remove a remote, run::

 $ flatpak remote-delete flathub

In this case, ``flathub`` is the remote's local name.

Installing applications
-----------------------

Search
``````

Applications can be found in any of your remotes using the ``search`` command. For example::

 $ flatpak search gimp

Search will return any applications matching the search terms. Each search result includes the application ID and the remote that the application is in. In this example, the search term is ``gimp``.

.. note::
  Search will only work for remotes whose application metadata has been updated. This can be done by either running ``flatpak update`` or ``flatpak update --appstream``.

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

Managing your applications
--------------------------

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
