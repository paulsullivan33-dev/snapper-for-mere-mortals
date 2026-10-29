# The Neighbours

By the end of this chapter you will know
what to reach for when snapper is not the
tool: Btrfs snapshots taken by hand,
`btrfs send` for moving snapshots to
safety, Timeshift, and plain backups. The
check: for each of the four, state the
job it does that snapper does not.

## Btrfs itself: snapshots by hand

Everything snapper does rests on one
Btrfs command, which you can run
yourself:

```
sudo btrfs subvolume snapshot -r / /.snapshots-manual/before-upgrade
```

That is a real snapshot — read-only,
copy-on-write, identical in mechanism to
snapper's. What it lacks is everything
around it: no number, no description, no
cleanup rule, no registry, no timeline,
no rollback choreography. For a one-off
"freeze this before I experiment", by
hand is honest and enough. For a system
that must protect itself every hour and
every package operation, forever,
unattended — that is a bookkeeping job,
and bookkeeping is snapper.

The Btrfs guide covers the filesystem
itself — subvolumes, snapshots,
`send`/`receive`, scrub and balance — and
this guide deliberately does not repeat
it: [Btrfs for Mere Mortals](https://paulsullivan33-dev.github.io/btrfs-for-mere-mortals/).

## `btrfs send`: the backup door snapper leaves open

A snapshot on the same disk is not a
backup (chapter 1). Btrfs's own answer is
`btrfs send`, which serialises a snapshot
— or just its difference from a parent
snapshot — to a stream another machine's
`btrfs receive` rebuilds. Snapper does
not wrap this; it is a deliberate seam.
The snapshots in `.snapshots` are
ordinary Btrfs snapshots, so a backup
routine can send them like any other.
Tools such as **btrbk** and **snapper's
usual companion scripts** exist precisely
in this seam: snapper minds the local
history, a sender ships it elsewhere.

## Timeshift

Timeshift is the neighbour most home
users meet first: a friendly graphical
tool promising system restore points. In
its Btrfs mode it manages snapshots of
the root subvolume on a schedule, with a
restore flow built for non-specialists.
Against snapper: Timeshift is narrower
(one layout, its own conventions about
subvolume names) and simpler; snapper is
broader (any subvolume, many configs,
pairs, userdata, a daemon API, scriptable
output) and plainer. They overlap on the
home ground — "undo a bad update" — and
either is defensible there. Running both
on the same subvolume is not: two
managers pruning one history is a mess
neither can see whole.

## Backup tools

Rsync-based tools, Borg, restic, and
their kin solve the other problem: copies
**elsewhere** — another disk, another
machine — that survive this machine's
death. They work on any filesystem, keep
their own history, and cannot boot you
back into last Tuesday's *system* the way
a rollback can. A complete setup usually
wants both kinds of protection: snapper
for change, a backup for loss. The
[Rsync guide](https://paulsullivan33-dev.github.io/rsync-for-mere-mortals/)
is this series' door into the second
kind.

## Where snapper refuses to go

Worth stating as its own fact: snapper
has no scheduling engine of its own
beyond the systemd timers, no network
code, no encryption, no off-site notion
at all. Each absence is a design choice
that keeps the tool small and auditable —
and each is filled, if you want it
filled, by a neighbour from this chapter
rather than by snapper growing.

## The check

Four jobs, four tools: freeze one moment
by hand — `btrfs subvolume snapshot`.
Keep a managed, self-pruning local
history — **snapper**. Move snapshots
off the machine — **`btrfs send`** (or a
tool built on it). Survive the machine's
death — a **backup**. If a plan uses one
tool for two of those jobs, it has a gap.
