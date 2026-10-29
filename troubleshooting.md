# Troubleshooting

Symptoms first, causes underneath. The
two error messages quoted below are the
lab's real transcripts — the failures the
lab *could* produce are the ones it hit
on purpose at the Btrfs boundary. The
rest are the documented failure shapes,
each with the check that separates it
from its neighbours.

## `create-config` says "Invalid subvolume."

The lab produced this one verbatim, exit
code 1, by pointing `create-config` at an
ordinary folder. The meaning is exact:
the path you named is **not the root of a
Btrfs subvolume** — either the filesystem
there is not Btrfs at all, or the path is
a directory *inside* a subvolume rather
than the subvolume itself. A subvolume's
root always has inode 256, so the check
you can run is:

```
stat -c %i /the/path
```

256 means a subvolume root; anything else
means keep looking — usually for the
subvolume's actual mount point.

## Snapper commands say the subvolume "is not a btrfs subvolume"

The lab's hand-written config was valid,
registered, and readable — and `list` and
`create` still failed:

```
IO error (query default id failed, subvolume is not a btrfs subvolume).
IO error (subvolume is not a btrfs subvolume).
```

Same family as above, one layer down: the
config parses fine, but the path in its
`SUBVOLUME` does not lead to a mounted
Btrfs subvolume. On a real machine the
usual cause is a subvolume that is not
mounted (or mounted somewhere else than
the config says) — check `findmnt` and
`btrfs subvolume list /` before suspecting
the config.

## `list-configs` shows nothing — but the config file exists

The lab demonstrated this one end to end:
a config file sitting in
`/etc/snapper/configs/` is **not enough**.
The config must also be named in
`SNAPPER_CONFIGS` in `/etc/default/snapper`.
Check the registry first:

```
grep SNAPPER_CONFIGS /etc/default/snapper
snapper list-configs
```

`create-config` maintains the registry;
hand-copied config files are where this
bites. The mirror-image fault — a name
registered whose file is gone — shows up
as errors naming the missing config.

## No timeline snapshots are appearing

Two switches and a timer, in check order:

1. The config: `TIMELINE_CREATE` must be
   `yes` — `snapper -c root get-config`.
2. The timer must be enabled:
   `systemctl status snapper-timeline.timer`.
3. The machine must have been **on**. The
   timeline records the hours the system
   lived through; gaps after a powered-off
   weekend are honest history, not a fault.

## No apt snapshots around installs

The hook's own preconditions, from the
file itself (chapter 9):

1. A config named **`root`** must exist —
   the hook checks for `/etc/snapper/configs/root`
   by name.
2. `DISABLE_APT_SNAPSHOT` in
   `/etc/default/snapper` must not be `yes`.
3. Remember the hook swallows errors
   (`|| true`), so a failing snapper never
   blocks dpkg — and never tells you. If
   the pair is missing, run the hook's own
   command by hand as root and read the
   error directly:
   `snapper create -d apt -c number -t pre -p`.

## The disk is filling with history

`snapper -c root list` and read the
used-space column: the oldest snapshots
with big numbers are holding deleted data
hostage. `cleanup all` first (chapter 7),
then delete deliberately. If used-space
shows nothing useful, quota is not set up
— `setup-quota` is what makes snapper able
to measure per-snapshot space at all. The
durable fixes are smaller limits and
ranged, space-aware limits, in chapter 7.

## A pre snapshot has no post

The operation it bracketed died before
finishing — a crashed installer, a killed
process, a power cut. The pre is harmless:
it is an ordinary snapshot with a story.
Leave it, delete it, or — if the change
half-happened — treat it as the "before"
picture for `status <pre>..0` while you
work out what state the system is in.

## Where snapper keeps its diary

`/var/log/snapper.log` records what
snapperd did and refused to do, including
cleanup decisions. When a snapshot vanished
"by itself", the log almost always shows
the algorithm, the rule, and the numbers
that condemned it. Bug reports are asked
to include this file; so should your own
debugging start with it.

## The check that ends most mysteries

```
snapper --version
snapper list-configs
snapper -c root get-config
```

Build flags, registered configs, and the
effective settings. Half the entries above
collapse into one line of one of those
three outputs.
