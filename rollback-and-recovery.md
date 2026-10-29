# Rollback and Recovering Files

By the end of this chapter you will know
four ways back, in increasing order of
drama: read a file out of a snapshot,
undo the changes between two snapshots,
restore with `undochange` for named
files, and roll the whole system back to
a snapshot and reboot into it. The check:
before any real emergency, walk the list
of your snapshots and say which one you
would roll back to, and why.

> **Lab note.** Every procedure in this
> chapter needs a mounted Btrfs system, so
> the chapter is **documented-only**,
> written from snapper's shipped manual —
> including the manual's own warnings,
> which are quoted rather than softened.

## Compare first: `status` and `diff`

Recovery starts with knowing what
changed. `status` lists the files that
differ between two snapshots:

```
snapper -c root status 4..7
```

Each line starts with a status string:
`+` for a file created, `-` for one
deleted, `c` for content changed, `t` for
the file's type changed, and further
letters for metadata — `p` permissions,
`u` user, `g` group, `x` extended
attributes. The range
syntax is `older..newer`, and `0` stands
for the live system, so `7..0` means
"everything that has changed since
snapshot 7". Paths in the filter files
(`/usr/share/snapper/filters/`) are
excluded, which keeps logs and caches
from burying the signal.

`diff` goes one level deeper and shows
the content differences, file by file:

```
snapper -c root diff 4..7 /etc/hosts
```

Leave the filename off to diff
everything — pipe it to a pager.

## The gentle way back: copy the file out

A snapshot is a folder tree. The single
most useful recovery in this guide needs
no snapper command at all:

```
ls /.snapshots/4/snapshot/etc/
cp /.snapshots/4/snapshot/etc/hosts /tmp/hosts.from-snapshot
```

Read it, compare it, copy it back when
you are sure. Nothing about the live
system changes until you copy — which is
why this is the method to reach for
first, in a calm moment, for the classic
accident: one file edited wrongly or
deleted. (If `SYNC_ACL` and the
`ALLOW_*` settings let your user read
`.snapshots`, you do not even need root.)

## Undo a range: `undochange`

```
sudo snapper -c root undochange 4..7
```

`undochange` works on the **live**
system: it makes the files that changed
between snapshot 4 and snapshot 7 look
the way they did in snapshot 4 — created
files removed, deleted files restored,
changed files reverted. Named files after
the range limit it to just those. This is
the surgical option: the system stays
booted, only the named stretch of history
is unwound.

Now the manual's warning, unsoftened:
**consistency after `undochange` is not
guaranteed.** Its own example: undo the
creation of a user, and files belonging
to that user may still exist. Undoing a
package upgrade file-by-file can leave
the package database describing a system
the files no longer match. `undochange`
restores files; it cannot restore the
*meaning* programs attached to them. For
package damage, the next section is the
honest tool.

## The full way back: `rollback`

```
sudo snapper -c root rollback 7
```

Rollback is how a Btrfs system undoes an
era. It does not edit the live system in
place. Instead it creates new snapshots —
a read-only one recording the current
state, and a read-write copy of snapshot
7 — and sets that copy as the
filesystem's **default subvolume**, the
one the machine boots from. Reboot, and
you are running snapshot 7 as your root.
The state you fled is still there as a
snapshot, so a rollback can itself be
rolled back.

Plain `rollback` with no number does the
mirror-image job: it snapshots the
current default and makes a fresh
read-write copy of the **currently running**
system the next boot target — the move
used after booting from a read-only
snapshot (for instance from a boot menu)
to make that state permanent.

Three facts keep rollback honest:

- It takes effect at **reboot**, not
  before. The running system is untouched
  until then.
- It works because the boot process
  mounts the default subvolume. A system
  whose bootloader or fstab names a
  specific subvolume instead can ignore
  the default — such layouts break
  rollback, and chapter 12 lists this as
  a precondition, not a detail.
- The build must carry the `rollback`
  flag. Chapter 3's check confirmed this
  guide's build does.

## Ordering the four ways back

Ask, in order: Is it **one file**? Copy
it out of the snapshot. Is it **a known
stretch of changes**, on a system that is
otherwise fine? `undochange`, with the
consistency warning in mind. Is it **the
system itself** — a bad upgrade, a broken
boot? `rollback` and reboot. And in every
case: `status` first, so the way back is
aimed at the change that actually hurt.

## The check

Run `snapper -c root status 1..0` on a
quiet machine and read the output. Every
line is a file that differs between your
first snapshot and now — which is also,
exactly, the list of what a rollback to
snapshot 1 would throw away.
