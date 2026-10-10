WINDOWS BUILD NOTES
====================

Below are some notes on how to build Megacoin Core for Windows.

The options known to work for building Megacoin Core on Windows are:

* On Linux using the [Mingw-w64](https://mingw-w64.org/doku.php) cross compiler tool chain.
* On Windows using [Windows Subsystem for Linux (WSL)](https://msdn.microsoft.com/commandline/wsl/about) and the Mingw-w64 cross compiler tool chain.

Installing Windows Subsystem for Linux
---------------------------------------

With Windows 10/11, Microsoft has released the [Windows Subsystem for Linux (WSL)](https://msdn.microsoft.com/commandline/wsl/about). This feature allows you to run a bash shell directly on Windows in an Ubuntu-based environment. Within this environment you can cross compile for Windows without the need for a separate Linux VM or server.

After the bash shell is active, you can follow the instructions below, starting with the "Cross-compilation" section. Compiling the 64-bit version is recommended.

Cross-compilation for Ubuntu and Windows Subsystem for Linux
------------------------------------------------------------

The steps below can be performed on Ubuntu (including in a VM) or WSL. The depends system
will also work on other Linux distributions, however the commands for installing the toolchain will be different.

First, install the general dependencies:

    sudo apt update
    sudo apt upgrade
    sudo apt install build-essential libtool autotools-dev automake pkg-config bsdmainutils curl git

A host toolchain (`build-essential`) is necessary because some dependency
packages (such as `protobuf`) need to build host utilities that are used in the
build process.

See also: [dependencies.md](dependencies.md).

## Building for 64-bit Windows

The first step is to install the mingw-w64 cross-compilation tool chain.

    sudo apt install g++-mingw-w64-x86-64

Set the default mingw32 g++ compiler option to posix:

    sudo update-alternatives --config x86_64-w64-mingw32-g++

Note that for WSL the Megacoin Core source path MUST be somewhere in the default mount file system, for example `/usr/src/megacoin`, AND not under `/mnt/d/`. If this is not the case the dependency autoconf scripts will fail.

Acquire the source in the usual way:

    git clone https://github.com/LIMXTEC/Megacoin.git

Once the source code is ready the build steps are below:

    PATH=$(echo "$PATH" | sed -e 's/:\/mnt.*//g') # strip out problematic Windows %PATH% imported var
    cd depends
    make HOST=x86_64-w64-mingw32
    cd ..
    ./autogen.sh
    CONFIG_SITE=$PWD/depends/x86_64-w64-mingw32/share/config.site ./configure --prefix=/
    make

## Building for 32-bit Windows

To build executables for Windows 32-bit, install the following dependencies:

    sudo apt install g++-mingw-w64-i686 mingw-w64-i686-dev
    sudo update-alternatives --config i686-w64-mingw32-g++  # Set the default mingw32 g++ compiler option to posix.

Acquire the source and build:

    git clone https://github.com/LIMXTEC/Megacoin.git
    PATH=$(echo "$PATH" | sed -e 's/:\/mnt.*//g')
    cd depends
    make HOST=i686-w64-mingw32
    cd ..
    ./autogen.sh
    CONFIG_SITE=$PWD/depends/i686-w64-mingw32/share/config.site ./configure --prefix=/
    make

## Depends system

For further documentation on the depends system see [README.md](../depends/README.md) in the depends directory.

Installation
-------------

After building using the Windows subsystem it can be useful to copy the compiled
executables to a directory on the Windows drive. This can be done in the following way:

    make install DESTDIR=/mnt/c/workspace/megacoin
