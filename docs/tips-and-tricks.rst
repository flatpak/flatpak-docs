Tips and Tricks
===============

This page explains a few useful features of the Flatpak CLI.


Testing an app with a different runtime
---------------------------------------

You can (for testing) run an application with a different runtime than it
typically uses.  For instance, to run stable gedit with the latest unstable
gnome runtime you can do::

 $ flatpak run --runtime-version=master org.gnome.gedit

You can also use a completely different runtime (but same version number)::

 $ flatpak run --runtime=org.gnome.Sdk org.gnome.gedit

 If you just want to use the sdk instead of the platform like the above, a
 better approach is to use ``-d``.

.. warning::

  Running against a runtime with a completely different ABI is undefined and unsupported
  behavior.

Downgrading
-----------

It is possible to install an older build of an application (or runtime).

First you look for the commit you are interested in::

 $ flatpak remote-info --log flathub org.gnome.Recipes

Then you deploy the commit::

 $ sudo flatpak update \
   --commit=ec07ad6c54e803d1428e5580426a41315e50a14376af033458e7a65bfb2b64f0 \
   org.gnome.Recipes

Note that currently it's not possible to pin an app to a commit, so if your
flatpaks are updated either manually or automatically, the downgraded app will
be included in the updates. See https://github.com/flatpak/flatpak/issues/3078

