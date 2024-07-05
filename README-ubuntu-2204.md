BPFTRACE und UBUNTU-22.04
=========================

Hier "meine" Beschreibung, wie ich BPFTRACE für Ubuntu baue.

Ich starte mit einem aktuellen, leeren Container von Ubuntu-22.04.

## Aktualisieren

Kommandos:

```
sudo apt update
sudo apt upgrade
```

Ausgaben:

```
ubuntu@bpftrace-2204-2:~$ sudo apt update
Hit:1 http://archive.ubuntu.com/ubuntu jammy InRelease
Get:2 http://archive.ubuntu.com/ubuntu jammy-updates InRelease [128 kB]
Hit:3 http://archive.ubuntu.com/ubuntu jammy-backports InRelease
Get:4 http://archive.ubuntu.com/ubuntu jammy-security InRelease [129 kB]
Fetched 257 kB in 1s (352 kB/s)  
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
All packages are up to date.

ubuntu@bpftrace-2204-2:~$ sudo apt upgrade
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
Calculating upgrade... Done
0 upgraded, 0 newly installed, 0 to remove and 0 not upgraded.
```

## Hilfsprogramme installieren

Kommandos:

```
# Nachfolgendes aus docker/Dockerfile.ubuntu
sudo apt install -y \
    asciidoctor \
    binutils-dev \
    bison \
    build-essential \
    clang \
    cmake \
    flex \
    libbpf-dev \
    libbpfcc-dev \
    libcereal-dev \
    libelf-dev \
    libiberty-dev \
    libpcap-dev \
    llvm-dev \
    liblldb-dev \
    libclang-dev \
    systemtap-sdt-dev \
    zlib1g-dev
sudo apt install devscripts
```

Ausgaben:

```
ubuntu@bpftrace-2204-2:~$ sudo apt install -y \
    asciidoctor \
    binutils-dev \
    bison \
    build-essential \
    clang \
    cmake \
    flex \
    libbpf-dev \
    libbpfcc-dev \
    libcereal-dev \
    libelf-dev \
    libiberty-dev \
    libpcap-dev \
    llvm-dev \
    liblldb-dev \
    libclang-dev \
    systemtap-sdt-dev \
    zlib1g-dev
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  binfmt-support binutils binutils-common binutils-x86-64-linux-gnu bzip2 clang-14 cmake-data cpp cpp-11 dh-elpa-helper dirmngr dpkg-dev emacsen-common fakeroot fontconfig-config fonts-dejavu-core fonts-lato g++ g++-11 gcc gcc-11 gcc-11-base gnupg gnupg-l10n gnupg-utils gpg gpg-agent gpg-wks-client gpg-wks-server
  gpgconf gpgsm icu-devtools javascript-common lib32gcc-s1 lib32stdc++6 libalgorithm-diff-perl libalgorithm-diff-xs-perl libalgorithm-merge-perl libarchive13 libasan6 libassuan0 libatomic1 libbinutils libbpfcc libbrotli1 libc-dev-bin libc-devtools libc6-dev libc6-i386 libcc1-0 libclang-14-dev libclang-common-14-dev
  libclang-cpp11 libclang-cpp14 libclang1-14 libcrypt-dev libctf-nobfd0 libctf0 libcurl4 libdbus-1-dev libdeflate0 libdpkg-perl libfakeroot libffi-dev libfile-fcntllock-perl libfl-dev libfl2 libfontconfig1 libfreetype6 libgc1 libgcc-11-dev libgd3 libgdbm-compat4 libgdbm6 libgomp1 libicu-dev libisl23 libitm1 libjbig0
  libjpeg-turbo8 libjpeg8 libjs-jquery libjsoncpp25 libksba8 libldap-2.5-0 libldap-common liblldb-14 liblldb-14-dev libllvm11 libllvm14 liblsan0 libmpc3 libmpfr6 libncurses-dev libnghttp2-14 libnpth0 libnsl-dev libobjc-11-dev libobjc4 libpcap0.8 libpcap0.8-dev libperl5.34 libpfm4 libpipeline1 libpng16-16 libquadmath0
  librhash0 librtmp1 libruby3.0 libsasl2-2 libsasl2-modules libsasl2-modules-db libsigsegv2 libssh-4 libstdc++-11-dev libtiff5 libtinfo-dev libtirpc-dev libtsan0 libubsan1 libuv1 libwebp7 libxml2-dev libxpm4 libz3-4 libz3-dev linux-libc-dev lldb-14 llvm llvm-14 llvm-14-dev llvm-14-linker-tools llvm-14-runtime
  llvm-14-tools llvm-runtime lto-disabled-list m4 make manpages manpages-dev patch perl perl-modules-5.34 pinentry-curses pkg-config python3-lldb-14 python3-pkg-resources python3-pygments python3-six rake rapidjson-dev rpcsvc-proto ruby ruby-asciidoctor ruby-net-telnet ruby-rubygems ruby-webrick ruby-xmlrpc ruby3.0
  rubygems-integration unzip xz-utils zip
...
Setting up ruby (1:3.0~exp1) ...
Setting up ruby-asciidoctor (2.0.16-2) ...
Setting up asciidoctor (2.0.16-2) ...

ubuntu@bpftrace-2204-2:~$ sudo apt install devscripts
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
...
update-perl-sax-parsers: Registering Perl SAX parser XML::SAX::Expat with priority 50...
update-perl-sax-parsers: Updating overall Perl SAX parser modules info file...
Replacing config file /etc/perl/XML/SAX/ParserDetails.ini with new version
Processing triggers for libc-bin (2.35-0ubuntu3.8) ...
```

## Quellverzeichnisse einbinden

Kommandos:

```
sed -e "s/^deb/deb-src/" /etc/apt/sources.list|sudo tee -a /etc/apt/sources.list
cat /etc/apt/sources.list
sudo apt update
sudo apt upgrade
```

Ausgaben:

```
ubuntu@bpftrace-2204-2:~$ sed -e "s/^deb/deb-src/" /etc/apt/sources.list|sudo tee -a /etc/apt/sources.list
deb-src http://archive.ubuntu.com/ubuntu jammy main restricted universe multiverse
deb-src http://archive.ubuntu.com/ubuntu jammy-updates main restricted universe multiverse
deb-src http://archive.ubuntu.com/ubuntu jammy-backports main restricted universe multiverse
deb-src http://archive.ubuntu.com/ubuntu jammy-security main restricted universe multiverse

ubuntu@bpftrace-2204-2:~$ cat /etc/apt/sources.list
deb http://archive.ubuntu.com/ubuntu jammy main restricted universe multiverse
deb http://archive.ubuntu.com/ubuntu jammy-updates main restricted universe multiverse
deb http://archive.ubuntu.com/ubuntu jammy-backports main restricted universe multiverse
deb http://archive.ubuntu.com/ubuntu jammy-security main restricted universe multiverse
deb-src http://archive.ubuntu.com/ubuntu jammy main restricted universe multiverse
deb-src http://archive.ubuntu.com/ubuntu jammy-updates main restricted universe multiverse
deb-src http://archive.ubuntu.com/ubuntu jammy-backports main restricted universe multiverse
deb-src http://archive.ubuntu.com/ubuntu jammy-security main restricted universe multiverse

ubuntu@bpftrace-2204-2:~$ sudo apt update
Hit:1 http://archive.ubuntu.com/ubuntu jammy InRelease
Hit:2 http://archive.ubuntu.com/ubuntu jammy-updates InRelease
Hit:3 http://archive.ubuntu.com/ubuntu jammy-backports InRelease
Hit:4 http://archive.ubuntu.com/ubuntu jammy-security InRelease
Get:5 http://archive.ubuntu.com/ubuntu jammy/restricted Sources [23.7 kB]
Get:6 http://archive.ubuntu.com/ubuntu jammy/universe Sources [17.8 MB]
Get:7 http://archive.ubuntu.com/ubuntu jammy/main Sources [1340 kB]
Get:8 http://archive.ubuntu.com/ubuntu jammy/multiverse Sources [304 kB]
Get:9 http://archive.ubuntu.com/ubuntu jammy-updates/main Sources [501 kB]
Get:10 http://archive.ubuntu.com/ubuntu jammy-updates/restricted Sources [67.2 kB]
Get:11 http://archive.ubuntu.com/ubuntu jammy-updates/multiverse Sources [19.5 kB]
Get:12 http://archive.ubuntu.com/ubuntu jammy-updates/universe Sources [341 kB]
Get:13 http://archive.ubuntu.com/ubuntu jammy-backports/main Sources [8144 B]
Get:14 http://archive.ubuntu.com/ubuntu jammy-backports/universe Sources [10.8 kB]
Get:15 http://archive.ubuntu.com/ubuntu jammy-security/multiverse Sources [10.5 kB]
Get:16 http://archive.ubuntu.com/ubuntu jammy-security/main Sources [279 kB]
Get:17 http://archive.ubuntu.com/ubuntu jammy-security/universe Sources [197 kB]
Get:18 http://archive.ubuntu.com/ubuntu jammy-security/restricted Sources [62.6 kB]
Fetched 21.0 MB in 4s (5496 kB/s)                       
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
All packages are up to date.

ubuntu@bpftrace-2204-2:~$ sudo apt upgrade
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
Calculating upgrade... Done
0 upgraded, 0 newly installed, 0 to remove and 0 not upgraded.
```

## Bauverzeichnis anlegen

Kommandos:

```
mkdir ~/build
```

## libbpf bauen

### Alte Version

Kommandos:

```
mkdir -p ~/build/libbpf
cd ~/build/libbpf
apt source libbpf-dev
sudo apt build-dep libbpf-dev
cd libbpf-0.5.0/
dpkg-buildpackage 
```

Ausgaben:

```
ubuntu@bpftrace-2204-2:~$ mkdir -p ~/build/libbpf
ubuntu@bpftrace-2204-2:~$ cd ~/build/libbpf

ubuntu@bpftrace-2204-2:~/build/libbpf$ apt source libbpf-dev
Reading package lists... Done
Picking 'libbpf' as source package instead of 'libbpf-dev'
NOTICE: 'libbpf' packaging is maintained in the 'Git' version control system at:
https://github.com/sudipm-mukherjee/libbpf.git
Please use:
git clone https://github.com/sudipm-mukherjee/libbpf.git
to retrieve the latest (possibly unreleased) updates to the package.
Need to get 825 kB of source archives.
Get:1 http://archive.ubuntu.com/ubuntu jammy-updates/main libbpf 0.5.0-1ubuntu22.04.1 (dsc) [1887 B]
Get:2 http://archive.ubuntu.com/ubuntu jammy-updates/main libbpf 0.5.0-1ubuntu22.04.1 (tar) [815 kB]
Get:3 http://archive.ubuntu.com/ubuntu jammy-updates/main libbpf 0.5.0-1ubuntu22.04.1 (diff) [8228 B]
Fetched 825 kB in 0s (2335 kB/s) 
dpkg-source: info: extracting libbpf in libbpf-0.5.0
dpkg-source: info: unpacking libbpf_0.5.0.orig.tar.gz
dpkg-source: info: unpacking libbpf_0.5.0-1ubuntu22.04.1.debian.tar.xz
dpkg-source: info: using patch list from debian/patches/series
dpkg-source: info: applying CVE-2022-3534.patch
dpkg-source: info: applying CVE-2022-3606.patch

ubuntu@bpftrace-2204-2:~/build/libbpf$ sudo apt build-dep libbpf-dev
Reading package lists... Done
Picking 'libbpf' as source package instead of 'libbpf-dev'
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following NEW packages will be installed:
  autoconf automake autopoint autotools-dev debhelper debugedit dh-autoreconf dh-strip-nondeterminism dwz libdebhelper-perl libdw1 libfile-stripnondeterminism-perl libsub-override-perl libtool po-debconf
0 upgraded, 15 newly installed, 0 to remove and 0 not upgraded.
Need to get 3201 kB of archives.
...

ubuntu@bpftrace-2204-2:~/build/libbpf$ cd libbpf-0.5.0/

ubuntu@bpftrace-2204-2:~/build/libbpf/libbpf-0.5.0$ dpkg-buildpackage 
dpkg-buildpackage: info: source package libbpf
dpkg-buildpackage: info: source version 0.5.0-1ubuntu22.04.1
dpkg-buildpackage: info: source distribution jammy-security
dpkg-buildpackage: info: source changed by Nishit Majithia <nishit.majithia@canonical.com>
...
dpkg-genbuildinfo -O../libbpf_0.5.0-1ubuntu22.04.1_amd64.buildinfo
 dpkg-genchanges -O../libbpf_0.5.0-1ubuntu22.04.1_amd64.changes
dpkg-genchanges: info: not including original source code in upload
 dpkg-source --after-build .
dpkg-buildpackage: info: binary and diff upload (original source NOT included)
```

### Aktualisierte Version

- Herunterladen: [libbpf-1.4.3.tar.gz](https://github.com/libbpf/libbpf/archive/refs/tags/v1.4.3.tar.gz)
- Virencheck
- Ablegen unter ~/build/libbpf/libbpf-1.4.3.tar.gz
- Bauverzeichnis aktualisieren:
  - `cd ~/build/libbpf/libbpf-0.5.0`
  - `uupdate -u ../libbpf-1.4.3.tar.gz`
  - `cd ../libbpf-1.4.3`
  - debian/changelog sichten/aktualisieren
  - `sed -i -e 's/^/#/' debian/patches/series`
- Paket bauen: `dpkg-buildpackage`

### Einspielen

```
cd ~/build/libbpf
sudo apt install ./libbpf-dev_1.4.3-1dp01~1ubuntu22.04.1~jammy_amd64.deb ./libbpf0_1.4.3-1dp01~1ubuntu22.04.1~jammy_amd64.deb
```

## libbpfcc bauen

### Alte Version

Kommandos:

```
mkdir -p ~/build/libbpfcc
cd ~/build/libbpfcc
apt source libbpfcc-dev
sudo apt build-dep libbpfcc-dev
cd bpfcc-0.18.0+ds/
dpkg-buildpackage
# scheitert -> ignorieren!
```

Ausgaben:

```
ubuntu@bpftrace-2204-2:~/build/libbpfcc/bpfcc-0.18.0+ds$ dpkg-buildpackage
...
[ 65%] Built target clang_frontend
make[2]: Leaving directory '/home/ubuntu/build/libbpfcc/bpfcc-0.18.0+ds/obj-x86_64-linux-gnu'
make[1]: *** [Makefile:149: all] Error 2
make[1]: Leaving directory '/home/ubuntu/build/libbpfcc/bpfcc-0.18.0+ds/obj-x86_64-linux-gnu'
dh_auto_build: error: cd obj-x86_64-linux-gnu && make -j16 "INSTALL=install --strip-program=true" VERBOSE=1 returned exit code 2
make: *** [debian/rules:18: binary] Error 2
dpkg-buildpackage: error: debian/rules binary subprocess returned exit status 2
```

### Aktualisierte Version

- Herunterladen und umbenennen: [bcc-src-with-submodule-0.30.0.tar.gz](https://github.com/iovisor/bcc/releases/download/v0.30.0/bcc-src-with-submodule.tar.gz)
- Virencheck
- Ablegen unter ~/build/libbpfcc/bcc-src-with-submodule-0.30.0.tar.gz
- Bauverzeichnis aktualisieren:
  - `cd ~/build/libbpfcc/bpfcc-0.18.0+ds`
  - `uupdate -u ../bcc-src-with-submodule-0.30.0.tar.gz`
  - `cd ../bpfcc-0.30.0`
  - debian/changelog sichten/aktualisieren
  - debian/control aktualisieren - python3-setuptools
  - debian/rules aktualisieren - 3 Zeilen mit 'swapin' deaktivieren
  - debian/patches/series anpassen
  
    ```
    fix-install-path.patch
    #0002-Specify-default-encoding-when-decoding-string-data.patch
    #2001_fix_path_to_deadloc.c.patch
    2002_fix_netqtop.c_path.patch
    #0001-cmake-link-dynamically-to-libclang-cpp-if-found-and-.patch
    #0002-cmake-always-link-to-packaged-libbpf-if-CMAKE_USE_LI.patch
    #0003-Remove-libbcc-no-libbpf-shared-library-change-libbcc.patch
    #0004-compat-defs.patch
    ```

- Zusatzpaket installieren: `sudo apt install python3-setuptools`
- Paket bauen: `dpkg-buildpackage`

### Einspielen

```
cd ~/build/libbpfcc
sudo apt install ./libbpfcc-dev_0.30.0-0dp01~jammy_amd64.deb ./libbpfcc_0.30.0-0dp01~jammy_amd64.deb
```

## bpftrace

```
mkdir -p ~/build/bpftrace
cd ~/build/bpftrace
apt source bpftrace
sudo apt build-dep bpftrace

cd bpftrace-0.14*
uupdate -u ../bpftrace-0.21.1....
cd ../bpftrace-0.21.1
# debian/changelog anpassen
# debian/rules anpassen: rmdir -> rm -rf
dpkg-buildpackage
```

## Historie

- 2024-07-04: Erste Version
