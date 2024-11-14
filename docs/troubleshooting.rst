Troubleshooting
===============

Sometimes you run into issues when building Flatpaks. This page includes a list
of common problems and how to solve them.

Remote server doesn't support shallow cloning
---------------------------------------------

If you get an error message like
``fatal: dumb http transport does not support shallow capabilities`` when
building your Flatpak application, it's because ``flatpak-builder`` will by
default attempt to shallow clone git repositories as an optimisation. However,
not all git remotes support shallow cloning, and in such cases you can disable
this optimisation by setting the ``disable-shallow-clone`` property to ``true``
in your source.
