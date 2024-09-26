# Minimal makefile for Sphinx documentation
#

# You can set these variables from the command line.
SPHINXOPTS    =
SPHINXBUILD  := sphinx-build
SPHINXPROJ    = Flatpak
SOURCEDIR     = .
BUILDDIR      = _build

ifeq ($(shell command -v $(SPHINXBUILD) 2> /dev/null),)
SPHINXBUILD := sphinx-build-3
endif

# Put it first so that "make" without argument is like "make help".
help:
	@$(SPHINXBUILD) -M help "$(SOURCEDIR)" "$(BUILDDIR)" $(SPHINXOPTS) $(O)

update-po:
	@$(SPHINXBUILD) -M gettext "$(SOURCEDIR)" "$(BUILDDIR)" $(SPHINXOPTS) $(O)
	@sphinx-intl update -p "$(BUILDDIR)/gettext" -d $(SOURCEDIR)/../po
	@potfiles=$$(find $(BUILDDIR)/gettext -name '*.pot' | sed 's|.*/||'); \
	pofiles=$$(find ../po -name '*.po' | sort -u); \
	for po in $$pofiles; do \
	  name=$$(echo $$po | sed 's|.*/LC_MESSAGES/||'); \
	  if ! [[ $$potfiles =~ $${name}t ]]; then \
	    rm $$po; \
	  fi; \
	done

.PHONY: help Makefile update-po

# Catch-all target: route all unknown targets to Sphinx using the new
# "make mode" option.  $(O) is meant as a shortcut for $(SPHINXOPTS).
%: Makefile
	@$(SPHINXBUILD) -M $@ "$(SOURCEDIR)" "$(BUILDDIR)" $(SPHINXOPTS) $(O)
