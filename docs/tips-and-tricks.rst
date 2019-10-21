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

It is possible to downgrade an installed application (or runtime) to an older
build.

First you look for the commit you are interested in::

 $ flatpak remote-info --log flathub org.gnome.Recipes

Then you deploy the commit::

 $ sudo flatpak update \
   --commit=ec07ad6c54e803d1428e5580426a41315e50a14376af033458e7a65bfb2b64f0 \
   org.gnome.Recipes

Note that currently it's not possible to pin an app to a commit, so if your
flatpaks are updated either manually or automatically, the downgraded app will
be included in the updates. See https://github.com/flatpak/flatpak/issues/3078

Bisecting regressions in application builds
-------------------------------------------

In case the newest builds of an application introduce regressions, you can use
``flatpak-bisect`` to discover which commit introduced the regression. It works
just like ``git bisect``.

In case your distribution doesn't install the ``flatpak-bisect`` utility, you can
find it distributed alongside the Flatpak source code, in https://github.com/flatpak/flatpak/blob/master/scripts/flatpak-bisect

First you update the application and get its history::

  $ flatpak-bisect org.gnome.gedit start

Then, you should set the current commit as the first bad commit::

  $ flatpak-bisect org.gnome.gedit bad

Now you need to find the hash of the first known good commit. For that, you can
see the build history by running::

  $ flatpak-bisect org.gnome.gedit log

To start bisecting, checkout the first known good commit you find::

  $ flatpak-bisect org.gnome.gedit checkout 5cd2b0648618c9038fbc6830733817309ade29541cdd8383830bbb76f6accf0d

After setting the bad commit and the first known good commit, you can launch
the application to verify if the current commit in the bisecting session is
a good or a bad one.

To mark a commit as good or bad, run::

  $ flatpak-bisect org.gnome.gedit good

Or::

  $ flatpak-bisect org.gnome.gedit bad

``flatpak-bisect`` will inform you when the first bad commit is found.
