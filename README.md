About the opensense tools
========================

In conjunction with src.git, ports.git, core.git and plugins.git they
create sets, packages and images for the opensense project.  The license
is a standard GPL v3 license.

Setting up a build system
=========================

Install FreeBSD 10.2-RELEASE (i386 or amd64 depending on your target arch)
on a machine with at least 10GB of hard disk (UFS works better than ZFS)
and at least 2GB of RAM.  All tasks require a root user.  Do the following
to grab the repositories (overwriting standard ports and src):

    # pkg install git
    # cd /usr
    # rm -rf src ports
    # git clone https://github.com/opensense/plugins
    # git clone https://github.com/opensense/ports
    # git clone https://github.com/opensense/tools
    # git clone https://github.com/opensense/core
    # git clone https://github.com/opensense/src
    # cd tools

TL;DR
=====

    # make everything

If successful, images can be found here: /tmp/images

Detailed build steps and options
================================

The build is broken down into individual stages: base,
kernel and ports can be built separately and repeatedly
without affecting the others.  All stages can be reinvoked
and continue building without cleaning the previous progress.
A final stage assembles all three stages into a target image.

All build steps are invoked via make(1):

    # make step OPTION="value"

Available build options are:

* CONFIG: 	reads the below from the specified file
* FLAVOUR:	"OpenSSL" (default), "LibreSSL"
* MIRRORS:	a list of mirrors to prefetch sets from
* NAME:		"opensense" (default)
* PRIVKEY:	the private key for signing sets
* PUBKEY:	the public key for signing sets
* SETTINGS:	the name of the selected settings in config/
* TYPE:         the name of the meta package to be installed
* VERSION:	a version tag (if applicable)

Build the userland binaries, bootloader and administrative
files:

    # make base

Build the kernel and loadable kernel modules:

    # make kernel

Build all the third-party ports:

    # make ports

Build additional plugins if needed:

    # make plugins

Wrap up our core as a package:

    # make core

A cdrom live image is created using:

    # make iso

A memstick image for VGA and serial is created using:

    # make memstick

A direct disk image in NanoBSD style is created using:

    # make nano

About other scripts and tweaks
==============================

Before building images, you can run the regression tests
to check the integrity of your core.git modifications plus
generate output for the style checker:

    # make regress

For very fast ports rebuilding of already installed packages
the following works:

    # make ports-<packagename>[,...]

For even faster ports building it may be of use to cache all
distribution files before running the actual build:

    # make distfiles

Compiled sets can be prefetched from a mirror, while removing
any previously available set:

    # make prefetch-<option>[,...] VERSION=version.to.prefetch

Available prefetch options are:

* base:		prefetch the base set
* kernel:	prefetch the kernel set
* packages:	prefetch the packages set

Core packages (pristine copies) can be batch-built using:

    # make core-<repo_branch_or_tag>[,...]

Package sets (may be signed depending on whether the key is
found under /root) ready for web server deployment are automatically
generated and modified by ports.sh and core.sh.

Release sets can be built using:

    # make release VERSION=product.version.number_revision

Kernel, base, packages and release sets are stored under /tmp/sets

All final images are stored under /tmp/images

A couple of build machine cleanup helpers are available
via the clean script:

    # make clean-<option>[,...]

Available clean options are:

* base:		remove the base set
* distfiles:	remove the distfiles set
* iso:		remove iso image
* kernel:	remove the kernel set
* memstick:	remove memstick images
* nano:		remove nano image
* packages:	remove the packages set
* release:	remove the release set
* src:		reset the kernel/base build directory
* stage:	reset the main staging area

The ports tree has a few of our modifications and is sometimes a
bit ahead of FreeBSD.  In order to keep the local changes, a skimming
script is used to review and copy upstream changes:

    # make skim[-<option>]

Available options are:

* used:		review and copy upstream changes
* unused:	copy unused upstream changes
* (none):	all of the above

In case a release was wrapped up, the base package list and obsoleted
files need to be regenerated.  This is done using:

    # make rebase

Shall any debugging be needed inside the build jail, the following
command will use chroot(8) to enter the active build jail:

    # make chroot
