flatpak-docs
============

This repository is the main source of developer documentation for Flatpak. It can be read at `flatpak.readthedocs.io <http://flatpak.readthedocs.io/>`_.

Some documentation is also available on the Flatpak wiki and as part of the ``flatpak`` and ``flatpak-builder`` man pages.

The docs are written in `reStructuredText <http://www.sphinx-doc.org/rest.html>`_ and contributions are welcome!

Building
--------

To build the docs locally, first install ``sphinx`` and ``sphinx_rtd_theme``.
On Fedora this can be with::

  sudo dnf install python3-sphinx python3-sphinx_rtd_theme

Then run ``make html`` in the ``docs`` directory.

Audience
--------

Desktop application developers are primary audience for the Flatpak docs, particularly the authors of existing popular and sought after applications.

The docs should reflect popular practice amongst this audience wherever possible, such as using Git for version control. Where specific tools and languages need to be featured, the most popular ones ought to be covered first.

The goal is to attract application developers who are not traditionally associated with the Linux desktop. Therefore, traditional Linux desktop conventions shoud not be assumed.

Guidelines
----------

Guidelines for those who want to contribute to the docs:

- Explain basic Flatpak concepts and standard application developer workflows
- Use the docs to explain the benefits of Flatpak and why a developer might use it
- Only cover what's essential for application developers - don't include details of Flatpak internals unless absolutely necessary
- Provide a developer experience that's as smooth and frictionless as possible
- Help to prevent difficulties by anticipating potential issues developers might hit, and steering them clear of them 
