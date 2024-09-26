flatpak-docs
============

This repository is the main source of developer documentation for Flatpak. It
can be read at `docs.flatpak.org <http://docs.flatpak.org/>`_.

Some documentation is also available on the Flatpak wiki and as part of the
``flatpak`` and ``flatpak-builder`` man pages.

The docs are written in `reStructuredText
<http://www.sphinx-doc.org/rest.html>`_ and contributions are welcome!

Setup Development
-----------------

Create a Python `virtual environment <https://docs.python.org/3/tutorial/venv.html#creating-virtual-environments>`_::

  python3 -m venv .venv && source .venv/bin/activate

Then install the dependencies with **pip**::

  pip install -r requirements.txt sphinx-intl

Build the Documentation
-----------------------

After setup, you can build the documentation::

  make -C docs html

Run the tests::

  make -C docs linkcheck

You can run an HTTP server and follow the printed link
(`localhost:8000 <http://localhost:8000>`_)
to view the documentation in your browser::

  python3 -m http.server -d docs/_build/html

Build Translations
------------------

By default, the document being built is in English. If you want to build
documents in other languages, such as Chinese, you can use the following
command::

  make -C docs html SPHINXOPTS='-D language=zh_CN'

Translate the Documentation
---------------------------

You can open a pull request adding a new language.

Maintainers can generate the template files (``.pot``), update the translation
files (``.po``) and remove obsolete translation files (i.e. a matching ``.pot``
file no longer exists) by running::

  make -C docs update-po

Audience
--------

Desktop application developers are the primary audience for the Flatpak
docs, particularly the authors of existing applications, including those
from non-Linux platforms.

The docs should reflect popular practice amongst this audience wherever
possible and not assume that applications are coming from the Linux desktop
space. In practical terms, this means that we should expect:

- Git for version control
- GitHub for hosting
- Freedesktop runtimes
- No prior knowledge of Linux desktop conventions, such as ``.desktop``
  files, AppStream and D-Bus

Outside of these basic defaults, special separate attention should be paid
to popular cross-platform technologies such as Electron and Qt.

Guidelines
----------

Guidelines for those who want to contribute to the docs:

- Explain basic Flatpak concepts
- Focus on standard application developer workflows
- Use the docs to explain the benefits of Flatpak and why a developer might
  use it
- Only cover what's essential for application developers - don't include
  details of Flatpak internals unless absolutely necessary
- Provide a developer experience that's as smooth and frictionless as possible
- Help to prevent difficulties by anticipating potential issues developers
  might hit, and steering them clear of them
