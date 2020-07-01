USB Drives
==========

One can distribute flatpaks along with their dependencies on USB drives (or
network shares, etc.) which is especially helpful in situations where Internet
access is limited or non-existent.

For offline distribution to work there are a few prerequisites:

- the remote repositories providing the app or any of its dependencies must
  utilize GPG signatures
- the remote repos must all have a collection ID set on the server side
- the locally configured remotes must have a collection ID set (on the client
  side)
- the relevant remotes must be configured on the receiving computer (the one
  installing from the drive)

Apps can then be copied to USB drives using `the flatpak create-usb command
<https://docs.flatpak.org/en/latest/flatpak-command-reference.html#flatpak-create-usb>`_.
You can refer to `this blog
post <https://blogs.gnome.org/mclasen/2018/08/26/about-flatpak-installations/>`__
for an introduction.

The process for installing from such a USB drive differs between Flatpak
versions before 1.8.0 and those after. With earlier versions you can simply use
the ``flatpak install`` command as you normally would online. For versions
after 1.8.0, if your linux distribution has packaged `the relevant systemd
units
<https://github.com/flatpak/flatpak/tree/master/sideload-repos-systemd>`__,
using ``flatpak install`` with no extra arguments still works. Otherwise, you
have to either use the ``--sideload-repo`` option in your command invocation or
create a symbolic link to the USB drive mount point in
``/run/flatpak/sideload-repos`` or a ``sideload-repos`` subdirectory in one of
your flatpak installations; see `the flatpak man
page <https://docs.flatpak.org/en/latest/flatpak-command-reference.html#flatpak>`__.

For example, if the output of ``df`` indicates that your USB drive is mounted
at ``/media/user/FLATPAKS``, you can execute these steps to make Flatpak treat
the USB as a local cache when doing installations or updates:

1. Create the directory in case it doesn't exist (you only need to do this
once). Enter your password when prompted::

   $ sudo sh -c 'mkdir -p /run/flatpak && mkdir -p -m 777 /run/flatpak/sideload-repos'

2. Create a symbolic link to the USB drive mount you found with the ``df``
command (replace both instances of ``/media/user/FLATPAKS`` here)::

   $ ln -s /media/user/FLATPAKS /run/flatpak/sideload-repos/`basename /media/user/FLATPAKS`

Then you are ready to use the ``flatpak install`` and ``flatpak update``
commands to install or update from the drive.
