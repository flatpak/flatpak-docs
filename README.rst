flatpak-docs
============

This repository is the main source of developer documentation for Flatpak. It
can be read at `flatpak.readthedocs.io <http://flatpak.readthedocs.io/>`_.

Some documentation is also available on the Flatpak wiki and as part of the
``flatpak`` and ``flatpak-builder`` man pages.

The docs are written in `reStructuredText
<http://www.sphinx-doc.org/rest.html>`_ and contributions are welcome!

Building
--------

To build the docs locally, first install ``sphinx`` and ``sphinx_rtd_theme``.
On Fedora this can be with::

  sudo dnf install python3-sphinx python3-sphinx_rtd_theme

On Debian this can be with::

  sudo apt install python3-sphinx python3-sphinx-rtd-theme

Then run ``make html`` in the ``docs`` directory.

By default, the document being built is in English. If you want to build
documents in other languages, such as Chinese, you can use the following
command::

  sphinx-build -b html -D language=zh_CN . _build/html/zh_CN

Then you will see the Chinese documentation in the directory
``_build/html/zh_CN`` .

Translations
------------

Translations are handled on `Zanata
<https://translate.zanata.org/project/view/flatpak-docs>`_.

You can open an issue requesting a new language added.

For maintainers run ``make gettext`` in the ``docs`` directory to generate
``.pot`` files.  To update ``.po`` files run ``sphinx-intl update -p
_build/gettext``

Audience
--------

Desktop application developers are the primary audience for the Flatpak docs,
particularly the authors of existing applications, including those from
non-Linux platforms.

The docs should reflect popular practice amongst this audience wherever
possible and not assume that applications are coming from the Linux desktop
space. In practical terms, this means that we should expect:

- Git for version control
- GitHub for hosting
- Freedesktop runtimes
- No prior knowledge of Linux desktop conventions, such as ``.desktop`` files,
  AppStream and D-Bus

Outside of these basic defaults, special separate attention should be paid to
popular cross-platform technologies such as Electron and Qt.

Guidelines
----------

Guidelines for those who want to contribute to the docs:

- Explain basic Flatpak concepts
- Focus on standard application developer workflows
- Use the docs to explain the benefits of Flatpak and why a developer might use
  it
- Only cover what's essential for application developers - don't include
  details of Flatpak internals unless absolutely necessary
- Provide a developer experience that's as smooth and frictionless as possible
- Help to prevent difficulties by anticipating potential issues developers
  might hit, and steering them clear of them
