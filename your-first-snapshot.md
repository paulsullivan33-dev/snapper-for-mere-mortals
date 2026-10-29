# Your First Snapshot

By the end of this chapter you will have
one config, one snapshot, and a list that
shows it. The check is the list itself:
one row, your description, today's date.

> **Lab note.** This chapter's procedure
> needs a mounted Btrfs subvolume, which
> the guide's lab could not mount (the
> reason is in `criteria.md`). The steps
> and outputs are snapper's documented
> behaviour, written from its manuals —
> **documented-only**, not a transcript.
> The config commands around the edges
> (`list-configs`, `get-config`) were run
> for real by the lab and are quoted from
> transcripts.

## Before you start: the subvolume must be a subvolume

`create-config` takes one argument: the
path of the subvolume to snapshot. For
the root filesystem that is `/` — on a
Btrfs system, `/` is the top of a
subvolume by construction. For a data
subvolume, it is that subvolume's mount
point, such as `/data`.

Snapper checks. A Btrfs subvolume has a
tell — its root folder always carries
inode number 256 — and snapper refuses a
path that fails the check. The lab
pointed `create-config` at an ordinary
folder on a non-Btrfs filesystem and got
the refusal verbatim, exit code 1:

```
Invalid subvolume.
```

If you meet that message on a real
machine, the path you named is not a
subvolume: either the filesystem there is
not Btrfs, or the folder is an ordinary
directory inside a subvolume rather than
the subvolume's root. Chapter 13 returns
to it.

## Create the config

```
sudo snapper -c root create-config /
```

Three things happen. A config file is
written to `/etc/snapper/configs/root`,
copied from the default template with
`SUBVOLUME="/"` filled in. The name
`root` is added to `SNAPPER_CONFIGS` in
`/etc/default/snapper`, which registers
it. And a `.snapshots` subvolume is
created inside `/`, ready to hold
snapshots.

Confirm the registration — this part the
lab did run, against its test root:

```
snapper list-configs
```

```
Config | Subvolume
-------+----------
root   | /
```

## Take the snapshot

```
sudo snapper -c root create --description "first snapshot"
```

It returns in about a second — remember
why: no data is copied. The snapshot
shares every block with the live system
and starts costing space only as files
change. Under the hood snapper has made
a read-only Btrfs snapshot at
`/.snapshots/1/snapshot` and written its
bookkeeping to `/.snapshots/1/info.xml`.

## Read the list

```
snapper -c root list
```

Snapshot **0** is listed by convention as
the current system. Your snapshot is
number 1, type `single` (chapter 7
explains the types), cleanup column
empty — a snapshot created by hand with
no `--cleanup-algorithm` is one snapper
will never delete for you — and your
description in the last column.

Add a second one after changing a file
or two, and notice the used-space column
start to tell the copy-on-write story:
the older snapshot now owns the old
versions of whatever you changed.

## The check

Run `snapper -c root list` and find your
description. Then run
`snapper -c root get-config` and meet the
config file as snapper sees it — chapter
6 walks every key in that table.
