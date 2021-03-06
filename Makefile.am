#
# Kimchi
#
# Copyright IBM Corp, 2013-2016
#
# This library is free software; you can redistribute it and/or
# modify it under the terms of the GNU Lesser General Public
# License as published by the Free Software Foundation; either
# version 2.1 of the License, or (at your option) any later version.
#
# This library is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the GNU
# Lesser General Public License for more details.
#
# You should have received a copy of the GNU Lesser General Public
# License along with this library; if not, write to the Free Software
# Foundation, Inc., 51 Franklin Street, Fifth Floor, Boston, MA  02110-1301  USA

SUBDIRS = contrib control distros.d docs model po tests ui xmlutils

kimchi_PYTHON = $(filter-out config.py, $(wildcard *.py))

nodist_kimchi_PYTHON = config.py

if WITH_SPICE
WITH_SPICE=yes
else
WITH_SPICE=no
endif

wokdir = $(pythondir)/wok
kimchidir = $(pythondir)/wok/plugins/kimchi

wokconfdir = $(sysconfdir)/wok/plugins.d
dist_wokconf_DATA = kimchi.conf

confdir = $(sysconfdir)/kimchi
dist_conf_DATA = template.conf

AUTOMAKE_OPTIONS = foreign

ACLOCAL_AMFLAGS = --install -I m4

EXTRA_DIST = \
	config.rpath \
	API.json \
	autogen.sh \
	COPYING.ASL2 \
	COPYING.LGPL \
	CONTRIBUTE.md \
	VERSION \
	build-aux/pkg-version \
	config.py.in \
	$(NULL)


PEP8_BLACKLIST = *config.py,*i18n.py,*tests/test_config.py

I18N_FILES = ./i18n.py \
	$(NULL)

check-local:
	contrib/check_i18n.py $(I18N_FILES)
	@if [ -d '.git' ]; then                                        \
		find . -path './.git' -prune -o                        \
			-name '*.py' -o -name '*.py.in'  |             \
			xargs $(PYFLAKES) |                            \
			while read LINE; do echo "$$LINE"; false; done \
	else                                                           \
		find . -name '*.py' -o -name '*.py.in'  |              \
			xargs $(PYFLAKES) |                            \
			while read LINE; do echo "$$LINE"; false; done \
	fi
	$(PEP8) --version
	$(PEP8) --filename '*.py,*.py.in' --exclude="$(PEP8_BLACKLIST)" .
	./check_ui_code_errors.sh
	@if $(GIT) rev-parse &> /dev/null ; then                         \
		echo "Whitespace verification ...";                      \
		git grep --cached -Il '' | grep -v '^ui/spice-html5/' |  \
			xargs egrep '.* +$$'                             \
			&& echo "ERROR: Whitespaces found" || echo "Ok"; \
		echo "IBM copyright year verification ..." ;              \
		/bin/bash ../../../../check-IBM-license-header.sh ;       \
	fi

# Link built mo files in the source tree to enable use of translations from
# within the source tree
all-local:
	while read L && test -n "$$L"; do                  \
		dir=mo/$$L/LC_MESSAGES ;                       \
		$(MKDIR_P) $$dir ;                             \
		ln -sf ../../../po/$$L.gmo $$dir/kimchi.mo ;   \
	done < po/LINGUAS

do_substitution = \
	sed -e 's,[@]prefix[@],$(prefix),g'             \
	-e 's,[@]datadir[@],$(datadir),g'               \
	-e 's,[@]sysconfdir[@],$(sysconfdir),g'         \
	-e 's,[@]localstatedir[@],$(localstatedir),g'   \
	-e 's,[@]pkgdatadir[@],$(pkgdatadir),g'         \
	-e 's,[@]wokdir[@],$(wokdir),g'           \
	-e 's,[@]kimchidir[@],$(kimchidir),g'           \
	-e 's,[@]kimchiversion[@],$(PACKAGE_VERSION),g' \
	-e 's,[@]kimchirelease[@],$(PACKAGE_RELEASE),g' \
	-e 's,[@]withspice[@],$(WITH_SPICE),g'

config.py: config.py.in Makefile
	$(do_substitution) < $(srcdir)/config.py.in > config.py


#
# Packaging helpers
#

install-deb: install
	cp -R $(top_srcdir)/contrib/DEBIAN $(DESTDIR)/
	mkdir -p $(DESTDIR)/$(localstatedir)/lib/kimchi
	mkdir -p $(DESTDIR)/$(localstatedir)/lib/kimchi/ws-tokens
	mkdir -p $(DESTDIR)/$(localstatedir)/lib/kimchi/screenshots
	mkdir -p $(DESTDIR)/$(localstatedir)/lib/kimchi/isos
	mkdir -p $(DESTDIR)/$(localstatedir)/lib/kimchi/virtviewerfiles


deb: contrib/make-deb.sh
	$(top_srcdir)/contrib/make-deb.sh

kimchi.spec: contrib/kimchi.spec.fedora contrib/kimchi.spec.suse
	@if test -e /etc/redhat-release; then                   \
		ln -sf contrib/kimchi.spec.fedora $@ ;              \
	elif test -e /etc/SuSE-release; then                    \
		ln -sf contrib/kimchi.spec.suse $@ ;                \
	else                                                    \
		echo "Unable to select a spec file for RPM build" ; \
		/bin/false ;                                        \
	fi

check_files:
	@if [ -d '.git' ]; then                                           \
               if [ `git status --porcelain --untracked-files=no | wc -l` -gt 0 ]; then        \
                       echo "***** Aborting 'make rpm' command *****";    \
                       echo "There are some changes not commited in your working \
directory. To avoid losing them, please commit or stage before run 'make rpm' command.";       \
                       exit 1;                                            \
               fi                                                         \
       fi

rpm: check_files dist kimchi.spec
	$(MKDIR_P) rpm/BUILD rpm/RPMS rpm/SOURCES rpm/SPECS rpm/SRPMS
	cp $(top_srcdir)/kimchi.spec rpm/SPECS/kimchi.spec
	cp $(DIST_ARCHIVES) rpm/SOURCES
	rpmbuild -ba --define "_topdir `pwd`/rpm" rpm/SPECS/kimchi.spec
	@if [ -d '.git' ]; then                         \
		git reset --hard;                       \
	fi

fedora-rpm: check_files contrib/kimchi.spec.fedora
	ln -sf contrib/kimchi.spec.fedora kimchi.spec
	$(MAKE) rpm

suse-rpm: check_files contrib/kimchi.spec.suse
	ln -sf contrib/kimchi.spec.suse kimchi.spec
	$(MAKE) rpm

ChangeLog:
	@if $(GIT) rev-parse &> /dev/null ; then                     \
		$(top_srcdir)/build-aux/genChangelog --release > $@; \
	fi

install-data-local:
	@if test -d "$(systemdsystemunitdir)" ; then \
		mkdir -p $(DESTDIR)/etc/systemd/system/wokd.service.d; \
		if test -f /etc/debian_version; then \
			$(INSTALL_DATA) contrib/kimchid.service.ubuntu $(DESTDIR)/etc/systemd/system/wokd.service.d/kimchi.conf; \
		else \
			$(INSTALL_DATA) contrib/kimchid.service.fedora $(DESTDIR)/etc/systemd/system/wokd.service.d/kimchi.conf; \
		fi; \
	fi; \
	$(MKDIR_P) $(DESTDIR)/$(localstatedir)/lib/kimchi/
	$(MKDIR_P) $(DESTDIR)$(kimchidir)
	$(INSTALL_DATA) API.json $(DESTDIR)$(kimchidir)/API.json
	mkdir -p $(DESTDIR)/$(localstatedir)/lib/kimchi/ws-tokens
	mkdir -p $(DESTDIR)/$(localstatedir)/lib/kimchi/screenshots
	mkdir -p $(DESTDIR)/$(localstatedir)/lib/kimchi/isos
	mkdir -p $(DESTDIR)/$(localstatedir)/lib/kimchi/virtviewerfiles

uninstall-local:
	@if test -f $(DESTDIR)/etc/systemd/system/wokd.service.d/kimchi.conf; then \
		$(RM) $(DESTDIR)/etc/systemd/system/wokd.service.d/kimchi.conf; \
	fi; \
	$(RM) $(DESTDIR)$(kimchidir)/API.json
	$(RM) -rf $(DESTDIR)/$(localstatedir)/lib/kimchi

VERSION:
	@if $(GIT) rev-parse &> /dev/null ; then                  \
		git describe --abbrev=0 --always > $@;            \
	fi

.PHONY: deb install-deb rpm fedora-rpm suse-rpm ChangeLog VERSION


clean-local:
	rm -rf mo rpm

BUILT_SOURCES = config.py
CLEANFILES = config.py kimchi.spec `find "$(top_srcdir)" -type f -name "*.pyc" -print`
