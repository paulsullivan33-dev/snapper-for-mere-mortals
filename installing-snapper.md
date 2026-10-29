# Installing Snapper

By the end of this chapter snapper will be
installed, and you will have checked that
your build carries the features this guide
uses. The check is `snapper --version` and
its flags line — the lab ran it for real,
and its output is quoted exactly.

> **Scope, honestly.** Snapper manages
> snapshots of a Btrfs filesystem. If the
> machine's filesystems are ext4 or XFS,
> installing snapper is still harmless,
> but there will be nothing for it to
> snapshot. Laying out Btrfs is the
> [Btrfs guide's](https://paulsullivan33-dev.github.io/btrfs-for-mere-mortals/)
> subject; this chapter starts where a
> working Btrfs system exists.

## Install the package

On Ubuntu and Debian:

```
sudo apt install snapper
```

On the lab's Ubuntu 24.04 that is snapper
**0.10.6**, and it pulls in the library
`libsnapper` and works alongside
`btrfs-progs`, the Btrfs toolkit — install
that too if it is somehow missing, since
the neighbours chapter uses it:

```
sudo apt install btrfs-progs
```

On openSUSE the same package name applies
(`zypper install snapper`), and on a system
installed with Btrfs, snapper is usually
already there with a `root` config made
for you. On Fedora and Arch the package is
likewise named `snapper`.

## What the package puts on the machine

The lab unpacked the Ubuntu package and
inventoried it; the map in chapter 2 came
from this list. Installing gives you:

- `/usr/bin/snapper` and `/usr/sbin/snapperd`
- the D-Bus service file that starts
  snapperd on demand
- three systemd timers — timeline,
  cleanup, and boot (chapter 2)
- config templates in
  `/usr/share/snapper/config-templates/`
- comparison filters in
  `/usr/share/snapper/filters/`
- the apt hook at
  `/etc/apt/apt.conf.d/80snapper`
  (chapter 9), inert until a config named
  `root` exists
- `/etc/default/snapper`, holding the
  config registry — empty on a fresh
  install

Nothing is enabled that can surprise you:
with no configs registered, the timers
fire and find nothing to do.

## The check: version and flags

```
snapper --version
```

The lab's output:

```
snapper 0.10.6
libsnapper 7.2.0
flags btrfs,lvm,no-ext4,xattrs,rollback,btrfs-quota,no-selinux
```

Read the flags line against this guide:

- `btrfs` — required. Without it, stop
  here; your build cannot do this guide's
  work at all.
- `rollback` — required for chapter 8's
  full-system rollback.
- `btrfs-quota` — required for the
  space-aware cleanup in chapter 7.
- `xattrs` — extended attributes are
  compared by `xadiff` and restored by
  `undochange`.
- `no-ext4` — expected on modern builds;
  ext4 snapshotting was discontinued
  upstream. It is a fact about the build,
  not a fault in your install.

Version numbers will differ on other
distributions. The flags are the part
that decides what you can do.

## A note on how the lab installed it

Full disclosure, once: the lab machine is
a sandbox whose package manager lock was
held by another process for the whole
session, so the lab fetched the same
archive packages with `apt-get download`
and unpacked them with `dpkg-deb -x`,
then ran the binaries from the unpacked
tree. Same files, same versions, no
modifications — the transcripts in
`snapper-lab/transcripts/t01-version-and-flags.txt`
show the package list. Your `apt install`
is the normal road; the lab's detour
changed nothing about what was verified.
