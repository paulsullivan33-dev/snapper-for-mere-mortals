# Snapper for Mere Mortals

Every Linux user knows the moment. An
update finishes, the machine reboots, and
something that worked yesterday does not
work today. The fix you want is not a
backup from last week. You want the
system as it was **ten minutes ago**,
before the change.

**Snapper** gives you that, on systems
that use the **Btrfs** filesystem. Snapper
is a snapshot manager written by SUSE.
It takes **snapshots** — frozen,
read-only copies of a filesystem folder
tree, made in a second because Btrfs
shares the data until something changes
it. Snapper takes them on a schedule,
takes them in pairs around package
operations (one before, one after), keeps
the number of them under control with
cleanup rules you set, and — the party
trick — can **roll the whole system
back**: reboot, and you are running the
snapshot, with the broken state kept
aside as a snapshot of its own.

This guide explains how snapper is put
together, how to install it and give it a
configuration, how to take and read
snapshots, how its three cleanup
algorithms decide what to throw away, and
how rollback and single-file recovery
work. The final chapters are honest
about the edges: a snapshot is not a
backup, a snapshot of a running database
is not a promise, and rollback only works
when the machine's layout was set up for
it.

The lab that wrote this guide has to
confess its limit up front, because the
series rule is that we never dress up
work we did not do. The lab ran the real
**snapper 0.10.6** from the Ubuntu 24.04
archives: the version and its build
options, the full command map, the whole
configuration layer against a test root
(configs written, registered, read back,
and changed), the package's timers and
its apt hook read line by line, and a
real Btrfs filesystem created and checked
with `btrfs-progs 6.6.3`. What the lab
could **not** do is mount that filesystem
— the sandbox withholds the device files
a mount needs — so no snapshot was ever
taken in the lab. Every procedure that
needs a mounted Btrfs filesystem is
written from snapper's own shipped
manuals and labelled **documented-only**
where it appears, and `criteria.md` keeps
the full ledger. The commands are the
real commands; the outputs described are
the documented ones, not transcripts.

## What this guide covers

- [**What Snapper is**](what-snapper-is.md) —
  the mental model: a snapshot is a frozen
  copy that costs almost nothing until
  files start to change
- [**The moving parts**](the-moving-parts.md) —
  the `snapper` command, the `snapperd`
  daemon behind D-Bus, configs, the
  `.snapshots` folder where snapshots
  live, and the timers
- [**Installing Snapper**](installing-snapper.md) —
  the package, what it brings with it, and
  the version check that proves which
  features your build has
- [**Your first snapshot**](your-first-snapshot.md) —
  one config, one snapshot, and how to
  read the list it lands in
- [**Configs and subvolumes**](configs-and-subvolumes.md) —
  one config per subvolume, the registry
  that turns config files on, and every
  setting in the file, in plain words
- [**Snapshot types and cleanup**](snapshot-types-and-cleanup.md) —
  single snapshots, pre/post pairs, the
  timeline, and the three cleanup
  algorithms that stop snapshots eating
  the disk
- [**Rollback and recovering files**](rollback-and-recovery.md) —
  comparing two snapshots, undoing the
  changes between them, pulling one file
  back by hand, and the full-system
  rollback
- [**Snapper and package managers**](snapper-and-package-managers.md) —
  the apt hook that brackets every package
  operation with a pre/post pair, the
  boot snapshot, and the hourly timeline
  service
- [**Recipes**](recipes.md) — snapshot
  before a risky edit, thin out a config
  that grew too fat, and rehearse a
  rollback on paper before you need one
- [**The neighbours**](the-neighbours.md) —
  Btrfs snapshots by hand, `btrfs send`
  for real backups, Timeshift, and why a
  snapshot is not a backup tool
- [**Limits**](limits.md) — no backup, no
  consistency promise for running
  programs, one filesystem per config,
  and rollback's layout demands
- [**Troubleshooting**](troubleshooting.md) —
  "Invalid subvolume", the config that
  exists but does not list, the timeline
  that never fires, and where the log
  lives
- [**Glossary**](glossary.md) — the terms,
  collected in one place

## What it doesn't cover

Setting up Btrfs itself — choosing it at
install time, subvolume layout, and
converting an existing system. That is
the [Btrfs guide's](https://paulsullivan33-dev.github.io/btrfs-for-mere-mortals/)
ground, and this guide points at it once
and then assumes a working Btrfs system.
LVM thin-provisioned snapshots, which
this build of snapper also supports, are
named but not taught. Sending snapshots
to another machine as backups is pointed
at, not taught — snapper deliberately
does not do it.

## How to use this guide

Read chapter 1 for the model and chapter
2 before installing anything — knowing
which part holds what turns later
confusion into a named problem. Chapter 3
installs, chapter 4 takes a first
snapshot in the same sitting. Chapters 5
and 6 are the ones to read slowly: the
cleanup settings are what stand between
you and a disk full of history. The lab's
transcripts live in
`snapper-lab/transcripts/`.

---

## Part of the series

This guide is one of the [For Mere Mortals](https://paulsullivan33-dev.github.io/mere-mortals/) guides: plain-language guides to the tools programmers and makers use.

Related guides:

- [Btrfs for Mere Mortals](https://paulsullivan33-dev.github.io/btrfs-for-mere-mortals/)
- [Rsync for Mere Mortals](https://paulsullivan33-dev.github.io/rsync-for-mere-mortals/)

## License

The writing in this guide is licensed under [CC BY 4.0](https://github.com/paulsullivan33-dev/snapper-for-mere-mortals/blob/main/LICENSE). Code examples and lab scripts are licensed under the [MIT License](https://github.com/paulsullivan33-dev/snapper-for-mere-mortals/blob/main/LICENSE-CODE).
