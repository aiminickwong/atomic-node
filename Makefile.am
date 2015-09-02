# Copyright (C) 2015 Red Hat, Inc.
#
# This program is free software; you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation; version 3 of the License.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with this program; if not, write to the Free Software
# Foundation, Inc., 51 Franklin Street, Fifth Floor, Boston,
# MA  02110-1301, USA.  A copy of the GNU General Public License is
# also available at http://www.gnu.org/copyleft/gpl.html.

PYTHONSOURCES = \
  $(shell find scripts -name \*.py)

SHELL := /bin/bash


EXTRA_DIST = \
  ovirt-node-atomic.spec.in \
  ovirt-node-atomic.spec

DISTCLEANFILES = $(PACKAGE)-$(VERSION).tar.gz \
	aclocal.m4 \
	configure \
	install-sh \
	missing \
	py-compile
	build/*.repo

DISTCLEANDIRS = autom4te.cache


#DISTRO = $(shell rpm --eval '%dist' | cut -c2-)
DISTRO = el7
TMPREPOS = tmp.repos
RPMBUILD_ARGS :=
RPMBUILD_ARGS += --define="_topdir `pwd`/$(TMPREPOS)"
RPMBUILD_ARGS += $(if $(RELEASE_SUFFIX), --define="release_suffix $$RELEASE_SUFFIX")
LPAREN:=(
RPAREN:=)
#REPOCMD := grep -h '\[' build/ovirt*.repo | \
#sed -e 's/^\[\$(LPAREN).*\$(RPAREN)\]$/"\1",/
#REPOCMD := grep -h '\[' build/ovirt*.repo  | sed -e 's/^\[\(.*\)\]$$/"\1",/g
REPOCMD := grep -h '\[' build/ovirt*.repo | sed -e 's/\[/"/' | sed -e 's/\]/",/'

srpm:	dist
	rm -fr "$(TMPREPOS)"
	mkdir -p $(TMPREPOS)/{SPECS,RPMS,SRPMS,SOURCES}
	$(RPMBUILD) $(RPMBUILD_ARGS) -ts "$(PACKAGE_TARNAME)-$(PACKAGE_VERSION).tar.gz"
	@echo
	@echo "srpm available at '$(TMPREPOS)'"
	@echo

rpm:	srpm
	$(RPMBUILD) $(RPMBUILD_ARGS) --rebuild "$(TMPREPOS)"/SRPMS/*.src.rpm
	@echo
	@echo "rpm(s) available at '$(TMPREPOS)'"
	@echo

# sed -i -e "s#@OVIRT_REPOS@#$(REPO)#" build/config.ini
# rm build/*.repo
repos: configs submodule
	cd ovirt-release/ovirt-release-3.6 && ./autogen.sh && ./configure
	cp ovirt-release/ovirt-release-3.6/ovirt.repo build/
	cp ovirt-release/ovirt-release-3.6/ovirt-$(DISTRO)-deps.repo build/
	sed -i -e 's/@DIST@/$(DISTRO)/' build/ovirt*.repo
	sed -ie 's/el7$$releasever/el7/' build/ovirt.repo
	cp build/ovirt.json.in build/ovirt.json
	$(eval REPOS := $(shell $(REPOCMD)))
	sed -i -e 's/@OVIRT_REPOS@/$(REPOS)/' build/ovirt.json
	sed -rie 's/", ]/" ]/' build/ovirt.json

configs:
	cp sig-atomic-buildscripts/*.repo build/
	cp sig-atomic-buildscripts/config.ini build/
	cp sig-atomic-buildscripts/centos-atomic-*.json build/
	cat build/ovirt.ini >> build/config.ini

submodule:
	git submodule update --init --recursive

snapshot-rpm:
	make rpm RELEASE_SUFFIX=".$$(date --utc +%Y%m%d).git$$(git rev-parse --short HEAD)"

publish:
	mkdir -p $(OVIRT_CACHE_DIR)
	rsync -aq $(TMPREPOS)/ $(OVIRT_CACHE_DIR)/ovirt
	createrepo $(OVIRT_CACHE_DIR)/ovirt


.PHONY: rpms publish srpms


check-local: doctests pep8 pyflakes pylint
	@echo -e "---\n Passed.\n---"

check-local-fast:
	@for M in $(PYTHONSOURCES); \
	do \
		export PARENT=$$$$ ; \
		echo "Fast checks on '$$M'"; \
		PYTHONPATH=. python -m doctest $$M || kill $$PARENT & \
		PYTHONPATH=. pep8 -r $$M || kill $$PARENT & \
		PYTHONPATH=. pyflakes $$M || kill $$PARENT & \
	done ; wait

doctests:
	@for M in $(PYTHONSOURCES); \
	do \
		echo Doctest on "$$M"; \
		PYTHONPATH=. python -m doctest $$M || exit 1; \
	done

pep8:
	@for M in $(PYTHONSOURCES); \
	do \
		echo pep8 on "$$M"; \
		PYTHONPATH=. pep8 -r $$M || exit 1; \
	done

PYLINT=pylint -f parseable --include-ids=yes --rcfile=.pylintrc
pylint:
	@for M in $(PYTHONSOURCES); \
	do \
		echo pylint on "$$M"; \
		PYTHONPATH=. $(PYLINT) $$M || exit 1; \
	done

pyflakes:
	@for M in $(PYTHONSOURCES); \
	do \
		echo pyflakes on "$$M"; \
		PYTHONPATH=. pyflakes $$M || exit 1; \
	done

# vim: ts=2
