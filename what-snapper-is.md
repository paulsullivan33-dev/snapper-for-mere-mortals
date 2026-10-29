# What Snapper Is

By the end of this chapter you will be
able to say what a snapshot is, why it is
nearly free to make, and what job snapper
does on top of it. The check: read the
last section and explain to yourself, in
one sentence, why a snapshot is not a
backup.

## The one-sentence version

Snapper manages **snapshots** of a
**subvolume** on a **Btrfs** filesystem:
it creates them by hand or on a schedule,
labels them, deletes the old ones by
rules you set, and can boot the machine
from one when the present goes wrong.

Three of those words carry the weight.

A **subvolume** is a folder tree inside a
Btrfs filesystem that the filesystem
treats as a unit — it can be mounted on
its own, and it can be snapshotted. Most
Btrfs Linux systems keep the whole
operating system in one subvolume,
usually named `@`, mounted as `/`.

A **snapshot** of a subvolume is a second
subvolume that starts out pointing at
exactly the same data. Btrfs is a
**copy-on-write** filesystem: when a
program changes a file in the live
system, Btrfs writes the new version to
fresh space and leaves the old blocks
where they were, still referenced by the
snapshot. So a snapshot costs a moment to
create and almost no disk — and then
grows, slowly, by exactly the amount of
data the live system has changed since.
Delete a snapshot and the blocks only it
referenced are freed.

**Snapper** is the manager SUSE wrote for
those snapshots. Btrfs itself can make a
snapshot with one command (the
[neighbours chapter](the-neighbours.md)
shows it). What Btrfs does not give you
is the bookkeeping: numbering, dates,
descriptions, before-and-after pairs,
automatic hourly snapshots, rules for
thinning them out, and a safe path back.
That bookkeeping is snapper's whole job.

## What a snapshot buys you

- **An undo for the whole system.** A
  package upgrade that breaks the machine
  can be reversed by rolling back to the
  snapshot taken just before it.
- **An undo for one file.** A snapshot is
  a real folder tree. A file you mangled
  or deleted is sitting in the snapshot,
  readable, copyable.
- **A before picture.** Comparing two
  snapshots lists every file that changed
  between them — the answer to "what did
  that update actually touch?"

## What a snapshot does not buy you

A snapshot lives **on the same filesystem
as the original**. If the disk dies, the
filesystem corrupts, or the machine is
stolen, the snapshots die with it. A
snapshot is protection against *changes*
— bad updates, bad edits, deleted files.
It is not protection against *loss*. That
is a backup's job, and snapper does not
pretend otherwise: it has no command to
copy a snapshot anywhere else. Chapter 11
names the tools that fill the gap.

The second limit is quieter. Snapper
freezes the filesystem's state at an
instant; it cannot ask running programs
to finish their writes first. The manual
says it plainly: there is no mechanism to
ensure the consistency of files while a
snapshot is made, so the files of a
running database can be inconsistent in
the snapshot. For config files and
package state — snapper's home ground —
that hardly matters. For live databases,
it does.

## Where snapper comes from

Snapper is an open-source project from
SUSE, written for openSUSE, where rolling
back a bad update from the boot menu is a
headline feature. It has since been
packaged for most distributions —
including the Ubuntu 24.04 package this
guide's lab examined, snapper 0.10.6. On
any distribution the ideas are the same;
what changes is how much is set up for
you. openSUSE installs with a snapper
config for the root filesystem already in
place. On Ubuntu you get the tools, the
timers, and an apt hook — but no config
until you make one, which is what chapter
4 does.

## The check

One sentence each:

- A snapshot protects against **change**,
  not **loss** — it shares a disk and a
  filesystem with the original.
- A snapshot costs **the data that has
  changed since it was taken**, not a full
  copy.
- Snapper's job is the **bookkeeping** —
  Btrfs does the freezing.
