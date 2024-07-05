BPFTRACE und UBUNTU
===================

Hier "meine" Beschreibung, wie ich BPFTRACE für Ubuntu baue.

Ubuntu-20.04
------------

Ich starte mit einem aktuellen, leeren Container von Ubuntu-22.04.

### Aktualisieren

Kommandos:

```
sudo apt update
sudo apt upgrade
```

### Hilfsprogramme installieren

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

### Quellverzeichnisse einbinden

Kommandos:

```
sed -e "s/^deb/deb-src/" /etc/apt/sources.list|sudo tee -a /etc/apt/sources.list
cat /etc/apt/sources.list
sudo apt update
sudo apt upgrade
```

### Bauverzeichnis anlegen

Kommandos:

```
mkdir ~/build
```

### libbpf bauen

#### Alte Version

Kommandos:

```
mkdir -p ~/build/libbpf
cd ~/build/libbpf
apt source libbpf-dev
sudo apt build-dep libbpf-dev
cd libbpf-0.5.0/
dpkg-buildpackage 
```

#### Aktualisierte Version

- Herunterladen: [libbpf-1.4.3.tar.gz](https://github.com/libbpf/libbpf/archive/refs/tags/v1.4.3.tar.gz)
- Virencheck
- Ablegen unter ~/build/libbpf/libbpf-1.4.3.tar.gz
- Bauverzeichnis aktualisieren:
  - `cd ~/build/libbpf/libbpf-0.5.0`
  - `uupdate -u ../libbpf-1.4.3.tar.gz`
  - `cd ../libbpf-1.4.3`
  - debian/changelog sichten/aktualisieren
  - ENTFÄLLT: `sed -i -e 's/^/#/' debian/patches/series`
- Paket bauen: `dpkg-buildpackage`
- Include-Dateien einspielen (nicht sauber! ): `sudo cp include/uapi/linux/* /usr/include/linux/.`
  - Gehören "eigentlich" zu linux-libc-dev
  - Sind zu alt!
  - Bspw. "bpf_link_type" fehlt!

#### Einspielen

```
cd ~/build/libbpf
sudo apt install ./libbpf-dev_1.4.3-1dp01~1ubuntu20.04.1~focal_amd64.deb ./libbpf0_1.4.3-1dp01~1ubuntu20.04.1~focal_amd64.deb 
```

### libbpfcc bauen

#### Alte Version

Kommandos:

```
mkdir -p ~/build/libbpfcc
cd ~/build/libbpfcc
apt source libbpfcc-dev
sudo apt build-dep libbpfcc-dev
#cd bpfcc-0.12.0
#dpkg-buildpackage
# scheitert -> ignorieren!
```

#### Aktualisierte Version

- Herunterladen und umbenennen: [bcc-src-with-submodule-0.30.0.tar.gz](https://github.com/iovisor/bcc/releases/download/v0.30.0/bcc-src-with-submodule.tar.gz)
- Virencheck
- Ablegen unter ~/build/libbpfcc/bcc-src-with-submodule-0.30.0.tar.gz
- Bauverzeichnis aktualisieren:
  - `cd ~/build/libbpfcc/bpfcc-0.12.0`
  - `uupdate -u ../bcc-src-with-submodule-0.30.0.tar.gz`
  - `cd ../bpfcc-0.30.0`
  - debian/changelog sichten/aktualisieren
  - debian/control aktualisieren - python3-setuptools
  - debian/rules aktualisieren
    - ENTFÄLLT: 3 Zeilen mit 'swapin' deaktivieren
    - Zeile mit 'rm -rf $(CURDIR)/src/cc/libbpf' entfernen deaktivieren
  - cmake/clang_libs.cmake anpassen

    ```diff
    --- bpfcc-0.30.0.orig/cmake/clang_libs.cmake
    +++ bpfcc-0.30.0/cmake/clang_libs.cmake
    @@ -68,7 +68,10 @@ if (${LLVM_PACKAGE_VERSION} VERSION_EQUA
     endif()
     
     list(APPEND clang_libs
    -  ${libclangBasic})
    +  ${libclangBasic}
    +  Polly
    +  PollyISL
    +)
    ```

  - debian/patches/series anpassen
  
    ```
    fix-install-path.patch
    #fix-ppc64.patch
    ```

- Zusatzpaket installieren: `sudo apt install python3-setuptools`
- Paket bauen: `dpkg-buildpackage`

#### Einspielen

```
cd ~/build/libbpfcc
sudo apt install ./libbpfcc-dev_0.30.0-0dp01~focal_amd64.deb ./libbpfcc_0.30.0-0dp01~focal_amd64.deb
```

### bpftrace

```
mkdir -p ~/build/bpftrace
cd ~/build/bpftrace
apt source bpftrace
sudo apt build-dep bpftrace

cd bpftrace-0.14*
uupdate -u ../bpftrace-0.21.1....
cd ../bpftrace-0.21.1
# debian/changelog anpassen
# debian/rules anpassen: Alles mit "man" weg
dpkg-buildpackage
```

### Test

```
uli@ulicsl:~$ sudo apt install ./bpftrace_0.21.1-1dp01~focal_amd64.deb

uli@ulicsl:~$ cd .../dptools

uli@ulicsl:~/git/dp-gitea/dptools$ sudo bpftrace bin/trace-open-close-rename.sh 
bpftrace: /usr/include/llvm-12/llvm/IR/Instructions.h:940: static llvm::GetElementPtrInst* llvm::GetElementPtrInst::Create(llvm::Type*, llvm::Value*, llvm::ArrayRef<llvm::Value*>, const llvm::Twine&, llvm::Instruction*): Assertion `PointeeType == cast<PointerType>(Ptr->getType()->getScalarType())->getElementType()' failed.
Abgebrochen
```

### Experimente mit LLVM-12 und CLANG-12

- Alle DEB-Pakete nochmal bauen mit LLVM-12 und CLANG-12 -> keine Änderung
- Zusätzlich: Optimierung abschalten -> keine Änderung

Links
-----

- [libbpf](https://github.com/libbpf/libbpf)
- [bcc - libbpfcc](https://github.com/iovisor/bcc/)
- [bpftrace](https://github.com/bpftrace/bpftrace/)

Historie
--------

- 2024-07-04: Erste Version
