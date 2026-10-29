# Limits

What snapper cannot do, stated plainly.
None of these are bugs; most are the
shape of the tool. All of them are
cheaper to learn here than in production.

## A snapshot is not a backup

The oldest limit, repeated from chapter 1
because everything depends on it: a
snapshot shares its disk and filesystem
with the original. Disk failure,
filesystem corruption, theft, and
`rm -rf` at the wrong level of the tree
take the snapshots with the system. If
snapper is your only protection, you are
protected against bad changes and nothing
else. Chapter 11 names the tools for the
rest.

## No consistency promise

Snapper freezes the filesystem at an
instant. It does not quiesce — pause and
flush — running programs first. The
manual is blunt: the files of a database
can be inconsistent while the database is
running. A snapshot of a live database is
a snapshot of a moment mid-sentence;
restoring it is like pulling the power
plug and hoping the journal saves you.
Databases and virtual machines need their
own snapshot cooperation, arranged with
those tools. Snapper's home ground —
system files, packages, configuration —
does not have this problem in practice,
because those files are not held open
mid-write by long-lived processes.

## Rollback has a layout precondition

Rollback works by changing the
filesystem's **default subvolume** and
rebooting. If the bootloader or
`/etc/fstab` mounts a *named* subvolume
instead of the default, the machine boots
the named one forever, and a rollback
silently changes nothing about what runs.
Distributions that set snapper up for you
arrange the layout it needs. Assembled by
hand, the layout is your responsibility —
and a rollback rehearsed in chapter 10 is
how you find out before it matters.

Rollback is also a **root filesystem**
story. Snapper can snapshot any
subvolume, but booting into a snapshot
only makes sense for the system one.

## Snapshots are not free, just cheap

A snapshot's cost is the data changed
since it was taken — but on a busy system
that is real space, held until the
snapshot is deleted. A month-old snapshot
of a system that updates daily can pin
gigabytes. Cleanup rules (chapter 7) are
not optional decoration; without them,
snapper's protection grows until the disk
fills, and a full Btrfs filesystem is its
own kind of outage. The used-space column
and quota-aware cleanup exist because
this limit bites in real life.

## One config, one subvolume, one filesystem

A config snapshots exactly one subvolume.
A system spread over several subvolumes
needs a config each, and they are
independent: there is no atomic snapshot
across subvolumes, so a rollback of `/`
does not rewind `/data`, even if both
were snapshotted in the same second. If
two trees must change together or not at
all, snapper is not the mechanism.

## The build decides some features

Rollback, quota support, and extended
attributes are compile-time options — the
flags line of chapter 3. This guide's
Ubuntu build has all three; a build
without them will refuse those operations
no matter how correct the command is.
Likewise, the Ubuntu build's `no-ext4`
flag is final: snapper's ext4 mode was
discontinued upstream, and no config will
talk this build into snapshotting ext4.
LVM thin-provisioned volumes are the only
other target this build supports, and
this guide does not teach them.

## undochange restores files, not meaning

Chapter 8 quoted the manual's warning;
it belongs here too as a limit.
`undochange` can revert files between two
snapshots, but programs keep state the
files only partly describe — package
databases, user databases, caches keyed
to files. An undone change can leave the
system inconsistent in ways no file
comparison will show. For package damage,
rollback is the sound tool.

## Smallprint from the manual

Descriptions and userdata should stick to
plain ASCII unless you are certain the
UTF-8 encoding is used everywhere — the
manual declines to promise otherwise.
Snapshot numbers and comparisons are
per-config, always. And the timeline can
only keep snapshots for hours the machine
was switched on: gaps in the timeline are
the truth about a laptop's life, not a
malfunction.
