Extensions
==========

In the introduction, Flatpak's key concepts were introduced: runtimes, bundled applications, SDKs, and sandboxes. There is one other concept that must be added to this list: extensions.

An extension is an optional piece that can be plugged into a filesystem, and they can be created for both applications and runtimes. Extensions are used for several purposes in Flatpak:

* To separate translations and debug information from each application. Since these parts can be large and are not always wanted, it makes sense to allow them to be optionally installed alongside the application.
* For software components that need to be distributed separately from an application or library. Primary examples of this include GL libraries (which need to be provided by the host) and GStreamer plugins (which can't always be distributed for legal reasons).

metadata
--------

Like runtimes, extensions are declared in the metadata of an application. They are then mounted by flatpak inside the application's sandbox. A typical extension section in a metadata file looks like this::

  [Extension org.gnome.Platform.GL]
  version=1.4
  directory=lib/GL

More complicated extension points can accept multiple extensions that are mounted below a single directory. For example, in this example the ``org.freedesktop.Platform`` runtime defines an extension point for GStreamer::

  [Extension org.freedesktop.Platform.GStreamer]
  version=1.4
  directory=lib/extensions/gstreamer-1.0
  subdirectories=true

Here, the ``subdirectories=true`` key means that an extension with the ``org.freedesktop.Platform.GStreamer.mp3`` ID would be mounted within ``/usr/lib/extensions/gstreamer-1.0/mp3`` in the sandbox. This is how the ``org.freedesktop.Platform`` knows where to look for GStreamer plugins.
