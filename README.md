[![gocryptfs](Documentation/gocryptfs-logo.png)](https://nuetzlich.net/gocryptfs/)
[![Build Status](https://travis-ci.org/rfjakob/gocryptfs.svg?branch=master)](https://travis-ci.org/rfjakob/gocryptfs)
[![MIT License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Go Report Card](https://goreportcard.com/badge/github.com/rfjakob/gocryptfs)](https://goreportcard.com/report/github.com/rfjakob/gocryptfs)

An encrypted overlay filesystem written in Go.
Official website: https://nuetzlich.net/gocryptfs  (Markdown [source](https://github.com/rfjakob/gocryptfs-website/tree/master/docs))

gocryptfs is built on top the excellent
[go-fuse](https://github.com/hanwen/go-fuse) FUSE library and its
LoopbackFileSystem API.

This project was inspired by EncFS and strives to fix its security
issues while providing good performance
([benchmarks](https://nuetzlich.net/gocryptfs/comparison/#performance)).

For details on the security of gocryptfs see the
[Security](https://nuetzlich.net/gocryptfs/security/) design document.

All tags from v0.4 onward are signed by the *gocryptfs signing key*.
Please check [Signed Releases](https://nuetzlich.net/gocryptfs/releases/)
for details.

Current Status
--------------

gocryptfs has reached version 1.0 on July 17, 2016. It has gone through
hours and hours of stress (fsstress, extractloop.bash) and correctness
testing (xfstests). It is now considered ready for general consumption.

The old principle still applies: Important data should have a backup.
Also, keep a copy of your master key (printed on mount) in a safe place.
This allows you to access the data even if the gocryptfs.conf config
file is damaged or you lose the password.

The security of gocryptfs has been audited in March 3, 2017. The audit
is available [here (defuse.ca)](https://defuse.ca/audits/gocryptfs.htm).

Platforms
---------

Linux is gocryptfs' native platform.

Beta-quality Mac OS X support is available, which means most things work
fine but you may hit an occasional problem. Check out
[ticket #15](https://github.com/rfjakob/gocryptfs/issues/15) for the history
of Mac OS X support but please create a new ticket if you a problem.

For Windows, an independent C++ reimplementation has been started:
[cppcryptfs](https://github.com/bailey27/cppcryptfs)

Testing
-------

gocryptfs comes with is own test suite that is constantly expanded as features are
added. Run it using `./test.bash`. It takes about 1 minute and requires FUSE
as it mounts several test filesystems.

The `stress_tests` directory contains stress tests that run indefinitely.

In addition, I have ported `xfstests` to FUSE, the result is the
[fuse-xfstests](https://github.com/rfjakob/fuse-xfstests) project. gocryptfs
passes the "generic" tests with one exception, results:  [XFSTESTS.md](Documentation/XFSTESTS.md)

A lot of work has gone into this. The testing has found bugs in gocryptfs
as well as in the go-fuse library.

Compile
-------

	$ go get -d github.com/rfjakob/gocryptfs
	$ cd $(go env GOPATH)/src/github.com/rfjakob/gocryptfs
	$ ./build.bash

build.bash needs the OpenSSL headers installed (Debian: `apt install libssl-dev`,
Fedora: `dnf install openssl-devel`). Alternatively, you can compile
without OpenSSL using

	$ ./build-without-openssl.bash

Use
---

	$ mkdir cipher plain
	$ ./gocryptfs -init cipher
	$ ./gocryptfs cipher plain

See the [Quickstart](https://nuetzlich.net/gocryptfs/quickstart/) page for more info.

The [MANPAGE.md](Documentation/MANPAGE.md) describes all available command-line options.

Graphical Interface
-------------------

The [SiriKali](https://mhogomchungu.github.io/sirikali/) project supports
gocryptfs and runs on Linux and OSX.

[cppcryptfs](https://github.com/bailey27/cppcryptfs) on Windows provides
its own GUI.

Stable CLI ABI
--------------

If you want to call gocryptfs from your app or script, see
[CLI_ABI.md](Documentation/CLI_ABI.md) for the official stable
ABI. This ABI is regression-tested by the test suite.

Storage Overhead
----------------

* Empty files take 0 bytes on disk
* 18 byte file header for non-empty files (2 bytes version, 16 bytes random file id)
* 32 bytes of storage overhead per 4kB block (16 byte nonce, 16 bytes auth tag)

[file-format.md](Documentation/file-format.md) contains a more detailed description.

Performance
-----------

Since version 0.7.2, gocryptfs is as fast as EncFS in the default mode,
and significantly faster than EncFS' "paranoia" mode that provides
a security level comparable to gocryptfs.

gocryptfs uses OpenSSL through a thin wrapper called `stupidgcm`.
This provides a 4x speedup compared to Go's builtin AES-GCM
implementation - see [openssl-gcm.md](Documentation/openssl-gcm.md)
for details. The use of openssl can disabled on the command-line.

Run `./benchmark.bash` to run gocryptfs' canonical set of
benchmarks that include streaming write, extracting a linux kernel
tarball, recursively listing and finally deleting it. The output will
look like this:

```
$ ./benchmark.bash
linux-3.0.tar.gz       100%[==========================>]  92,20M  2,96MB/s    in 35s
2016-05-04 19:29:20 URL:https://www.kernel.org/pub/linux/kernel/v3.0/linux-3.0.tar.gz
WRITE: 131072000 bytes (131 MB) copied, 1,43137 s, 91,6 MB/s
UNTAR: 23.25
LS:    1.75
RM:    4.42
```

Changelog
---------

v1.4.2, 2017-11-01
* Add `Gopkg.toml` file for `dep` vendoring and reproducible builds
  ([issue #142](https://github.com/rfjakob/gocryptfs/issues/142))
* MacOS: deal with `.DS_Store` files inside CIPHERDIR
  ([issue #140](https://github.com/rfjakob/gocryptfs/issues/140))
* Reverse mode: fix ENOENT error affecting names exactly 176 bytes long
  ([issue #143](https://github.com/rfjakob/gocryptfs/issues/143))
* Support kernels compiled with > 128 kiB FUSE request size (Synology NAS)
  ([issue #145](https://github.com/rfjakob/gocryptfs/issues/145))
* Fix a startup hang when `$PATH` contains the mountpoint
  ([issue #146](https://github.com/rfjakob/gocryptfs/issues/146))

v1.4.1, 2017-08-21
* **Use memory pools for buffer handling** (
  [3c6fe98](https://github.com/rfjakob/gocryptfs/commit/3c6fe98),
  [b2a23e9](https://github.com/rfjakob/gocryptfs/commit/b2a23e9),
  [12c0101](https://github.com/rfjakob/gocryptfs/commit/12c0101))
  * On my machine, this **doubles** the streaming read speed
    (see [performance.txt](https://github.com/rfjakob/gocryptfs/blob/v1.4.1/Documentation/performance.txt#L38))
* Implement and use the getdents(2) syscall for a more efficient
  OpenDir implementation
  ([e50a6a5](https://github.com/rfjakob/gocryptfs/commit/e50a6a5))
* Purge masterkey from memory as soon as possible
  ([issue #137](https://github.com/rfjakob/gocryptfs/issues/137))
* Reverse mode: fix inode number collision between .name and .diriv
  files
  ([d12aa57](https://github.com/rfjakob/gocryptfs/commit/d12aa57))
* Prevent the logger from holding stdout open
  ([issue #130](https://github.com/rfjakob/gocryptfs/issues/130))
* MacOS: make testing without openssl work properly
  ([ccf1a84](https://github.com/rfjakob/gocryptfs/commit/ccf1a84))
* MacOS: specify a volume name
  ([9f8e19b](https://github.com/rfjakob/gocryptfs/commit/9f8e19b))
* Enable writing to write-only files
  ([issue #125](https://github.com/rfjakob/gocryptfs/issues/125))

v1.4, 2017-06-20
* **Switch to static binary releases**
  * From gocryptfs v1.4, I will only release statically-built binaries.
    These support all Linux distributions but cannot use OpenSSL.
  * OpenSSL is still supported - just compile from source!
* Add `-force_owner` option to allow files to be presented as owned by a
  different user or group from the user running gocryptfs. Please see caveats
  and guidance in the man page before using this functionality.
* Increase open file limit to 4096 ([#82](https://github.com/rfjakob/gocryptfs/issues/82)).
* Implement path decryption via ctlsock ([#84](https://github.com/rfjakob/gocryptfs/issues/84)).
  Previously, decryption was only implemented for reverse mode. Now both
  normal and reverse mode support both decryption and encryption of
  paths via ctlsock.
* Add more specific exit codes for the most common failure modes,
  documented in [CLI_ABI.md](Documentation/CLI_ABI.md)
* Reverse mode: make sure hard-linked files always return the same
  ciphertext
  ([commit 9ecf2d1a](https://github.com/rfjakob/gocryptfs/commit/9ecf2d1a3f69e3d995012073afe3fc664bd928f2))
* Display a shorter, friendlier help text by default.
* **Parallelize file content encryption** by splitting data blocks into two
  threads ([ticket#116](https://github.com/rfjakob/gocryptfs/issues/116))
* Prefetch random nonces in the background
  ([commit 80516ed](https://github.com/rfjakob/gocryptfs/commit/80516ed3351477793eec882508969b6b29b69b0a))
* Add `-info` option to pretty-print infos about a filesystem.

v1.3, 2017-04-29
* **Use HKDF to derive separate keys for GCM and EME**
  * New feature flag: `HKDF` (enabled by default)
  * This is a forwards-compatible change. gocryptfs v1.3 can mount
   filesystems created by earlier versions but not the other way round.
* Enable Raw64 filename encoding by default (gets rid of trailing `==` characters)
* Drop Go 1.4 compatibility. You now need Go 1.5 (released 2015-08-19)
  or higher to build gocryptfs.
* Add `-serialize_reads` command-line option
  * This can greatly improve performance on storage
    that is very slow for concurrent out-of-order reads. Example:
    Amazon Cloud Drive ([#92](https://github.com/rfjakob/gocryptfs/issues/92))
* Reject file-header-only files
  ([#90 2.2](https://github.com/rfjakob/gocryptfs/issues/90),
  [commit](https://github.com/rfjakob/gocryptfs/commit/14038a1644f17f50b113a05d09a2a0a3b3e973b2))
* Increase max password size to 2048 bytes ([#93](https://github.com/rfjakob/gocryptfs/issues/93))
* Use stable 64-bit inode numbers in reverse mode
  * This may cause problems for very old 32-bit applications
    that were compiled without Large File Support.
* Passing "--" now also blocks "-o" parsing

v1.2.1, 2017-02-26
* Add an integrated speed test, `gocryptfs -speed`
* Limit password size to 1000 bytes and reject trailing garbage after the newline
* Make the test suite work on [Mac OS X](https://github.com/rfjakob/gocryptfs/issues/15)
* Handle additional corner cases in `-ctlsock` path sanitization
* Use dedicated exit code 12 on "password incorrect"

v1.2, 2016-12-04
* Add a control socket interface. Allows to encrypt and decrypt filenames.
  For details see [backintime#644](https://github.com/bit-team/backintime/issues/644#issuecomment-259835183).
  * New command-line option: `-ctlsock`
* Under certain circumstances, concurrent truncate and read could return
  an I/O error. This is fixed by introducing a global open file table
  that stores the file IDs
  ([commit](https://github.com/rfjakob/gocryptfs/commit/0489d08ae21107990d0efd0685443293aa26b35f)).
* Coalesce 4kB ciphertext block writes up to the size requested through
  the write FUSE call
  ([commit with benchmarks](https://github.com/rfjakob/gocryptfs/commit/024511d9c71558be4b1169d6bb43bd18d65539e0))
* Add `-noprealloc` command-line option
  * Greatly speeds up writes on Btrfs
    ([#63](https://github.com/rfjakob/gocryptfs/issues/63))
    at the cost of reduced out-of-space robustness.
  * This is a workaround for Btrfs' slow fallocate(2)
* Preserve owner for symlinks an device files (fixes bug [#64](https://github.com/rfjakob/gocryptfs/issues/64))
* Include rendered man page `gocryptfs.1` in the release tarball

v1.1.1, 2016-10-30
* Fix a panic on setting file timestamps ([go-fuse#131](https://github.com/hanwen/go-fuse/pull/131))
* Work around an issue in tmpfs that caused a panic in xfstests generic/075
  ([gocryptfs#56](https://github.com/rfjakob/gocryptfs/issues/56))
* Optimize NFS streaming writes
  ([commit](https://github.com/rfjakob/gocryptfs/commit/a08d55f42d5b11e265a8617bee16babceebfd026))

v1.1, 2016-10-19
* **Add reverse mode ([#19](https://github.com/rfjakob/gocryptfs/issues/19))**
  * AES-SIV (RFC5297) encryption to implement deterministic encryption
    securely. Uses the excellent
    [jacobsa/crypto](https://github.com/jacobsa/crypto) library.
    The corresponding feature flag is called `AESSIV`.
  * New command-line options: `-reverse`, `-aessiv`
  * Filesystems using reverse mode can only be mounted with gocryptfs v1.1
    and later.
  * The default, forward mode, stays fully compatible with older versions.
    Forward mode will keep using GCM because it is much faster.
* Accept `-o foo,bar,baz`-style options that are passed at the end of
  the command-line, like mount(1) does. All other options must still
  precede the passed paths.
  * This allows **mounting from /etc/fstab**. See
    [#45](https://github.com/rfjakob/gocryptfs/issues/45) for details.
  * **Mounting on login using pam_mount** works as well. It is
    [described in the wiki](https://github.com/rfjakob/gocryptfs/wiki/Mounting-on-login-using-pam_mount).
* To prevent confusion, the old `-o` option had to be renamed. It is now
  called `-ko`. Arguments to `-ko` are passed directly to the kernel.
* New `-passfile` command-line option. Provides an easier way to read
  the password from a file. Internally, this is equivalent to
  `-extpass "/bin/cat FILE"`.
* Enable changing the password when you only know the master key
  ([#28](https://github.com/rfjakob/gocryptfs/issues/28))

v1.0, 2016-07-17
* Deprecate very old filesystems, stage 3/3
  * Filesystems created by v0.6 can no longer be mounted
  * Drop command-line options `-gcmiv128`, `-emenames`, `-diriv`. These
    are now always enabled.
* Add fallocate(2) support
* New command-line option `-o`
  * Allows to pass mount options directly to the kernel
* Add support for device files and suid binaries
  * Only works when running as root
  * Must be explicitly enabled by passing "-o dev" or "-o suid" or "-o suid,dev"
* Experimental Mac OS X support. See
  [ticket #15](https://github.com/rfjakob/gocryptfs/issues/15) for details.

v0.12, 2016-06-19
* Deprecate very old filesystems, stage 2/3
  * Filesystems created by v0.6 and older can only be mounted read-only
  * A [message](https://github.com/rfjakob/gocryptfs/blob/v0.12/internal/configfile/config_file.go#L120)
    explaining the situation is printed as well
* New command line option: `-ro`
  * Mounts the filesystem read-only
* Accept password from stdin as well ([ticket #30](https://github.com/rfjakob/gocryptfs/issues/30))

v0.11, 2016-06-10
* Deprecate very old filesystems, stage 1/3
  * Filesystems created by v0.6 and older can still be mounted but a
    [warning](https://github.com/rfjakob/gocryptfs/blob/v0.11/internal/configfile/config_file.go#L120)
    is printed
  * See [ticket #29](https://github.com/rfjakob/gocryptfs/issues/29) for details and
    join the discussion
* Add rsync stress test "pingpong-rsync.bash"
  * Fix chown and utimens failures that caused rsync to complain
* Build release binaries with Go 1.6.2
  * Big speedup for CPUs with AES-NI, see [ticket #23](https://github.com/rfjakob/gocryptfs/issues/23)

v0.10, 2016-05-30
* **Replace `spacemonkeygo/openssl` with `stupidgcm`**
  * gocryptfs now has its own thin wrapper to OpenSSL's GCM implementation
    called `stupidgcm`.
  * This should fix the [compile issues](https://github.com/rfjakob/gocryptfs/issues/21)
    people are seeing with `spacemonkeygo/openssl`. It also gets us
    a 20% performance boost for streaming writes.
* **Automatically choose between OpenSSL and Go crypto** [issue #23](https://github.com/rfjakob/gocryptfs/issues/23)
  * Go 1.6 added an optimized GCM implementation in amd64 assembly that uses AES-NI.
    This is faster than OpenSSL and is used if available. In all other
    cases OpenSSL is much faster and is used instead.
  * `-openssl=auto` is the new default
  * Passing `-openssl=true/false` overrides the autodetection.
* Warn but continue anyway if fallocate(2) is not supported by the
  underlying filesystem, see [issue #22](https://github.com/rfjakob/gocryptfs/issues/22)
  * Enables to use gocryptfs on ZFS and ext3, albeit with reduced out-of-space safety.
* [Fix statfs](https://github.com/rfjakob/gocryptfs/pull/27), by @lxp
* Fix a fsstress [failure](https://github.com/hanwen/go-fuse/issues/106)
  in the go-fuse library.

v0.9, 2016-04-10
* **Long file name support**
  * gocryptfs now supports file names up to 255 characters.
  * This is a forwards-compatible change. gocryptfs v0.9 can mount filesystems
   created by earlier versions but not the other way round.
* Refactor gocryptfs into multiple "internal" packages
* New command-line options:
  * `-longnames`: Enable long file name support (default true)
  * `-nosyslog`: Print messages to stdout and stderr instead of syslog (default false)
  * `-wpanic`: Make warning messages fatal (used for testing)
  * `-d`: Alias for `-debug`
  * `-q`: Alias for `-quiet`

v0.8, 2016-01-23
* Redirect output to syslog when running in the background
* New command-line option:
  * `-memprofile`: Write a memory allocation debugging profile the specified
    file

v0.7.2, 2016-01-19
* **Fix performance issue in small file creation**
  * This brings performance on-par with EncFS paranoia mode, with streaming writes
    significantly faster
  * The actual [fix](https://github.com/hanwen/go-fuse/commit/c4b6b7949716d13eec856baffc7b7941ae21778c)
    is in the go-fuse library. There are no code changes in gocryptfs.

v0.7.1, 2016-01-09
* Make the `build.bash` script compatible with Go 1.3
* Disable fallocate on OSX (system call not availabe)
* Introduce pre-built binaries for Fedora 23 and Debian 8

v0.7, 2015-12-20
* **Extend GCM IV size to 128 bit from Go's default of 96 bit**
  * This pushes back the birthday bound to make IV collisions virtually
    impossible
  * This is a forwards-compatible change. gocryptfs v0.7 can mount filesystems
    created by earlier versions but not the other way round.
* New command-line option:
  * `-gcmiv128`: Use 128-bit GCM IVs (default true)

v0.6, 2015-12-08
* **Wide-block filename encryption using EME + DirIV**
  * EME (ECB-Mix-ECB) provides even better security than CBC as it fixes
    the prefix leak. The used Go EME implementation is
    https://github.com/rfjakob/eme which is, as far as I know, the first
    implementation of EME in Go.
  * This is a forwards-compatible change. gocryptfs v0.6 can mount filesystems
    created by earlier versions but not the other way round.
* New command-line option:
  * `-emenames`: Enable EME filename encryption (default true)

v0.5.1, 2015-12-06
* Fix a rename regression caused by DirIV and add test case
* Use fallocate to guard against out-of-space errors

v0.5, 2015-12-04
* **Stronger filename encryption: DirIV**
  * Each directory gets a random 128 bit file name IV on creation,
    stored in `gocryptfs.diriv`
  * This makes it impossible to identify identically-named files across
    directories
  * A single-entry IV cache brings the performance cost of DirIV close to
    zero for common operations (see performance.txt)
  * This is a forwards-compatible change. gocryptfs v0.5 can mount filesystems
    created by earlier versions but not the other way round.
* New command-line option:
  * `-diriv`: Use the new per-directory IV file name encryption (default true)
  * `-scryptn`: allows to set the scrypt cost parameter N. This option
    can be used for faster mounting at the cost of lower brute-force
    resistance. It was mainly added to speed up the automated tests.

v0.4, 2015-11-15
* New command-line options:
  * `-plaintextnames`: disables filename encryption, added on user request
  * `-extpass`: calls an external program for prompting for the password
  * `-config`: allows to specify a custom gocryptfs.conf path
* Add `FeatureFlags` gocryptfs.conf paramter
  * This is a config format change, hence the on-disk format is incremented
  * Used for ext4-style filesystem feature flags. This should help avoid future
    format changes. The first user is `-plaintextnames`.
* On-disk format 2

v0.3, 2015-11-01
* **Add a random 128 bit file header to authenticate file->block ownership**
  * This is an on-disk-format change
* On-disk format 1

v0.2, 2015-10-11
* Replace bash daemonization wrapper with native Go implementation
* Better user feedback on mount failures

v0.1, 2015-10-07
* First release
* On-disk format 0
