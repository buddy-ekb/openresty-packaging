### Makefile ---

## Author: shell@xps13
## Version: $Id: Makefile,v 0.0 2017/03/22 04:56:03 shell Exp $
## Keywords:
## X-URL:

# you need to install the libtemplate-perl package first to get the tpage utility.
# we use Perl TT2 templates to generate */debian/changelog files.

JOBS=8
ZLIB_VER=1.2.11
PCRE_VER=8.42
OR_VER=1.13.6.2
LEMPLATE_VER=0.15
TEST_NGINX_VER=0.26
OPTS=
DISTRO:=$(shell lsb_release -c|awk '{print $$2}')

.PHONY: build
build: zlib-build pcre-build \
	openresty-build openresty-debug-build \
	openresty-valgrind-build \
	lemplate-build test-nginx-build

.PHONY: clean
clean: zlib-clean pcre-clean \
	openresty-clean openresty-debug-clean openresty-valgrind-clean \
	lemplate-clean test-nginx-clean

.PHONY: zlib-download
zlib-download:
	rm -f *.orig.tar.*
	wget -nH --cut-dirs=100 --mirror 'http://www.zlib.net/fossils/zlib-$(ZLIB_VER).tar.gz'
	cp zlib-$(ZLIB_VER).tar.gz openresty-zlib_$(ZLIB_VER).orig.tar.gz

.PHONY: zlib-build
zlib-build: | zlib-clean zlib-download
	rm -f *.deb *.debian.tar.xz *.dsc *.changes
	tar xf openresty-zlib_$(ZLIB_VER).orig.tar.gz --strip-components=1 -C openresty-zlib
	cd openresty-zlib \
		&& tpage --define distro=$(DISTRO) debian/changelog.tt2 > debian/changelog \
		&& debuild $(OPTS) -j$(JOBS)
	#for f in *.deb; do debsigs --sign=origin -K D5EDEB74 $$f || exit 1; done
	if [ -f ./upload ]; then ./upload || exit 1; fi

.PHONY: zlib-clean
zlib-clean:
	cd openresty-zlib && debclean
	find openresty-zlib -maxdepth 1 ! -name 'debian' ! -name 'openresty-zlib' -print | xargs rm -rf
	rm -f openresty-zlib*.deb
	rm -f openresty-zlib_*.*
	#rm -f zlib-$(ZLIB_VER).tar.gz

.PHONY: pcre-download
pcre-download:
	rm -f *.orig.tar.*
	wget -nH --cut-dirs=100 --mirror 'https://ftp.pcre.org/pub/pcre/pcre-$(PCRE_VER).tar.bz2'
	cp pcre-$(PCRE_VER).tar.bz2 openresty-pcre_$(PCRE_VER).orig.tar.bz2

.PHONY: pcre-build
pcre-build: | pcre-clean pcre-download
	rm -f *.deb *.debian.tar.xz *.dsc *.changes
	tar xf openresty-pcre_$(PCRE_VER).orig.tar.bz2 --strip-components=1 -C openresty-pcre
	cd openresty-pcre \
		&& tpage --define distro=$(DISTRO) debian/changelog.tt2 > debian/changelog \
		&& debuild $(OPTS) -j$(JOBS)
	#for f in openresty-pcre*.deb; do debsigs --sign=origin -K D5EDEB74 $$f || exit 1; done
	if [ -f ./upload ]; then ./upload || exit 1; fi

.PHONY: pcre-clean
pcre-clean:
	cd openresty-pcre && debclean
	find openresty-pcre -maxdepth 1 ! -name 'debian' ! -name 'openresty-pcre' -print | xargs rm -rf
	rm -f openresty-pcre*.deb
	rm -f openresty-pcre_*.*
	#rm -f pcre-$(PCRE_VER).tar.bz2

.PHONY: openresty-download
openresty-download:
	rm -f *.orig.tar.*
	wget -nH --cut-dirs=100 --mirror 'https://openresty.org/download/openresty-$(OR_VER).tar.gz'
	cp openresty-$(OR_VER).tar.gz openresty_$(OR_VER).orig.tar.gz

.PHONY: openresty-build
openresty-build: | openresty-clean openresty-download
	sudo apt-get -y -qq install openresty-zlib-dev openresty-pcre-dev libssl-dev
	rm -f *.deb *.debian.tar.xz *.dsc *.changes
	tar xf openresty_$(OR_VER).orig.tar.gz --strip-components=1 -C openresty
	cd openresty \
		&& cp -pr ../../../nginx-ssl-ja3 bundle/nginx-ssl-ja3-0.0 \
		&& patch < ../../openresty-configure-module-nginx-ssl-ja3.patch \
		&& ( cd bundle/nginx-1.13.6 && patch -p1 < ../../../../../nginx-ssl-ja3/docker/debian-nginx-ssl-ja3/nginx.ssl.extensions.patch ) \
		&& tpage --define distro=$(DISTRO) debian/changelog.tt2 > debian/changelog \
		&& dpkg-buildpackage -b
#		&& debuild $(OPTS) -j$(JOBS)
	#for f in openresty*.deb; do debsigs --sign=origin -K D5EDEB74 $$f || exit 1; done
	if [ -f ./upload ]; then ./upload || exit 1; fi

.PHONY: openresty-clean
openresty-clean:
	sudo apt-get -y -qq install openresty-zlib-dev openresty-pcre-dev libssl-dev
	cd openresty && debclean
	find openresty -maxdepth 1 ! -name 'debian' ! -name 'openresty' -print | xargs rm -rf
	rm -f openresty*.deb
	rm -f openresty_*.*

.PHONY: openresty-debug-download
openresty-debug-download:
	rm -f *.orig.tar.*
	wget -nH --cut-dirs=100 --mirror 'https://openresty.org/download/openresty-$(OR_VER).tar.gz'
	cp openresty-$(OR_VER).tar.gz openresty-debug_$(OR_VER).orig.tar.gz

.PHONY: openresty-debug-build
openresty-debug-build: | openresty-debug-clean openresty-debug-download
	sudo apt-get -y -qq install openresty-zlib-dev openresty-pcre-dev
	rm -f *.deb *.debian.tar.xz *.dsc *.changes
	tar xf openresty-debug_$(OR_VER).orig.tar.gz --strip-components=1 -C openresty-debug
	cd openresty-debug \
		&& tpage --define distro=$(DISTRO) debian/changelog.tt2 > debian/changelog \
		&& debuild -sa $(OPTS) -j$(JOBS)
	#for f in openresty-debug*.deb; do debsigs --sign=origin -K D5EDEB74 $$f || exit 1; done
	if [ -f ./upload ]; then ./upload || exit 1; fi

.PHONY: openresty-debug-clean
openresty-debug-clean:
	sudo apt-get -y -qq install openresty-zlib-dev openresty-pcre-dev
	-cd openresty-debug && debclean
	find openresty-debug -maxdepth 1 ! -name 'debian' ! -name 'openresty-debug' -print | xargs rm -rf
	rm -f openresty-debug*.deb
	rm -f openresty-debug_*.*

.PHONY: openresty-valgrind-download
openresty-valgrind-download:
	rm -f *.orig.tar.*
	wget -nH --cut-dirs=100 --mirror 'https://openresty.org/download/openresty-$(OR_VER).tar.gz'
	cp openresty-$(OR_VER).tar.gz openresty-valgrind_$(OR_VER).orig.tar.gz

.PHONY: openresty-valgrind-build
openresty-valgrind-build: | openresty-valgrind-clean openresty-valgrind-download
	sudo apt-get -y -qq install openresty-zlib-dev openresty-pcre-dev valgrind
	rm -f *.deb *.debian.tar.xz *.dsc *.changes
	tar xf openresty-valgrind_$(OR_VER).orig.tar.gz --strip-components=1 -C openresty-valgrind
	cd openresty-valgrind \
		&& tpage --define distro=$(DISTRO) debian/changelog.tt2 > debian/changelog \
		&& debuild -sa $(OPTS) -j$(JOBS)
	#for f in openresty-valgrind*.deb; do debsigs --sign=origin -K D5EDEB74 $$f || exit 1; done
	if [ -f ./upload ]; then ./upload || exit 1; fi

.PHONY: openresty-valgrind-clean
openresty-valgrind-clean:
	sudo apt-get -y -qq install openresty-zlib-dev openresty-pcre-dev valgrind
	-cd openresty-valgrind && debclean
	find openresty-valgrind -maxdepth 1 ! -name 'debian' ! -name 'openresty-valgrind' -print | xargs rm -rf
	rm -f openresty-valgrind*.deb
	rm -f openresty-valgrind_*.*

.PHONY: lemplate-download
lemplate-download:
	rm -f *.orig.tar.*
	wget -nH --cut-dirs=100 --mirror 'http://search.cpan.org/CPAN/authors/id/A/AG/AGENT/Lemplate-$(LEMPLATE_VER).tar.gz'
	cp Lemplate-$(LEMPLATE_VER).tar.gz liblemplate-perl_$(LEMPLATE_VER).orig.tar.gz

.PHONY: lemplate-clean
lemplate-clean:
	-cd liblemplate-perl/ && debclean
	find liblemplate-perl -maxdepth 1 ! -name 'debian' ! -name 'liblemplate-perl' -print | xargs rm -rf
	rm -f liblemplate-perl*.deb
	rm -f liblemplate-perl_*.*

.PHONY: lemplate-build
lemplate-build: | lemplate-clean lemplate-download
	sudo apt-get -y -qq install libipc-run3-perl libfile-find-rule-perl libtext-glob-perl libnumber-compare-perl
	rm -f *.deb *.debian.tar.xz *.dsc *.changes
	tar xf liblemplate-perl_$(LEMPLATE_VER).orig.tar.gz --strip-components=1 -C liblemplate-perl
	cd liblemplate-perl \
		&& tpage --define distro=$(DISTRO) debian/changelog.tt2 > debian/changelog \
		&& debuild -sa $(OPTS) -j$(JOBS)
	#for f in openresty-valgrind*.deb; do debsigs --sign=origin -K D5EDEB74 $$f || exit 1; done
	if [ -f ./upload ]; then ./upload || exit 1; fi

.PHONY: test-nginx-download
test-nginx-download:
	rm -f *.orig.tar.*
	wget -nH --cut-dirs=100 --mirror 'http://search.cpan.org/CPAN/authors/id/A/AG/AGENT/Test-Nginx-$(TEST_NGINX_VER).tar.gz'
	cp Test-Nginx-$(TEST_NGINX_VER).tar.gz libtest-nginx-perl_$(TEST_NGINX_VER).orig.tar.gz

.PHONY: test-nginx-clean
test-nginx-clean:
	-cd libtest-nginx-perl/ && debclean
	find libtest-nginx-perl -maxdepth 1 ! -name 'debian' ! -name 'libtest-nginx-perl' -print | xargs rm -rf
	rm -f libtest-nginx-perl*.deb
	rm -f libtest-nginx-perl_*.*

.PHONY: test-nginx-build
test-nginx-build: | test-nginx-clean test-nginx-download
	sudo apt-get -y -qq install libtest-base-perl libtest-longstring-perl libtext-diff-perl liblist-moreutils-perl
	rm -f *.deb *.debian.tar.xz *.dsc *.changes
	tar xf libtest-nginx-perl_$(TEST_NGINX_VER).orig.tar.gz --strip-components=1 -C libtest-nginx-perl
	cd libtest-nginx-perl \
		&& tpage --define distro=$(DISTRO) debian/changelog.tt2 > debian/changelog \
		&& debuild -sa $(OPTS) -j$(JOBS)
	#for f in openresty-valgrind*.deb; do debsigs --sign=origin -K D5EDEB74 $$f || exit 1; done
	if [ -f ./upload ]; then ./upload || exit 1; fi

### Makefile ends here
