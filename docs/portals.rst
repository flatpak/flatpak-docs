Portal support in GTK
=====================

GTK will transparently use portals for some functionality when it detects that
it is being used inside a Flatpak sandbox. Here are some hints for what GTK
applications should do to benefit from this.

- Use ``g_get_user_config_dir()``, ``g_get_user_cache_dir()`` and
  ``g_get_user_data_dir()`` to find the right place to store configuration
  and data
- Use ``GtkFileChooserNative`` (or ``GtkFileChooserButton``) to open
  files. As of `xdg-desktop-portal-gtk` 1.7.1 it can also open directories.
- Use ``GtkPrintOperation`` for printing
- Use ``gtk_show_uri_on_window()`` or ``g_app_info_launch_default_for_uri()``
  to open URIs
- Use ``gtk_application_inhibit()`` if you want to inhibit idle or logout
- Use ``g_application_send_notification()`` to show notifications
- Use the ``GtkApplication::screensaver-active`` property to monitor
  scrensaver status

Additionally, xdg-desktop-portal-gtk is required for GTK themes and fonts.
It is also used as a fallback for the filepicker if the main xdg-desktop-portal implementation doesn't support filepicking.

Portal support in Qt and KDE
=============================

Qt and KDE libraries will transparently use portals for some functionality when
they detect that they are being used inside a Flatpak sandbox. Here are some
hints for what Qt or KDE applications should do to benefit from this.

- Use ``QDesktopServices::openUrl(const QUrl &url)`` or ``KIO::OpenUrlJob`` to
  open URIs or send an email when using ``mailto`` URL
- Use ``QFileDialog`` class to open files and, as of Qt ``5.18.90``, directories. Avoid using
  ``QFileDialog::DontUseNativeDialog``.
- Use ``KNotification::notify()`` to show notification
