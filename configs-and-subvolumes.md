# Configs and Subvolumes

By the end of this chapter you will be
able to read a snapper config file top to
bottom and change it safely. The check:
run `get-config` on your config and
explain, in plain words, what its three
cleanup limits are currently set to.

> **Lab note.** The config layer is the
> part of snapper the lab verified most
> deeply. It wrote a config from the
> shipped template, watched it stay
> invisible to `list-configs` until it was
> registered, read all 23 keys back with
> `get-config`, and changed two limits
> with `set-config`, confirming the
> rewritten file. Transcript:
> `snapper-lab/transcripts/t04-fake-root-configs.txt`.
> Only the snapshot operations in that
> transcript failed — at the Btrfs
> boundary, exactly where this chapter
> says the config layer ends.

## One config per subvolume

A config answers for exactly one
subvolume. A typical machine has one:
`root`, answering for `/`. A machine with
a separate Btrfs subvolume for data might
have two: `root` for `/`, `data` for
`/data`. Each has its own snapshot folder,
its own limits, its own timeline switch.
Configs never share snapshots, and a
cleanup run on one never touches another.

Which subvolumes deserve a config? The
ones whose **change history** you would
want to rewind: the operating system
first, always. A subvolume of media
files that only grows is usually better
served by a backup tool — snapshots of it
protect against almost nothing and are
covered in chapter 11's terms.

## The registry: a file is not enough

Configs live as files in
`/etc/snapper/configs/`, but snapper only
works with the ones named in
`SNAPPER_CONFIGS` in `/etc/default/snapper`:

```
SNAPPER_CONFIGS="root"
```

`create-config` and `delete-config`
maintain that line for you. If you ever
copy a config file into place by hand —
restoring one from a backup, say — add
its name to the registry too, or every
snapper command will act as if it does
not exist. The lab demonstrated this the
plain way: same file, invisible before
registration, fully listed after.

## Reading the file

`get-config` prints the effective
settings as a table. The lab's run against
a template-fresh config showed 23 keys.
They group into five families:

**What and where**

- `SUBVOLUME` — the path this config
  snapshots. Set at creation; `set-config`
  cannot change it, or `FSTYPE`.
- `FSTYPE` — `btrfs` for this guide.

**Space**

- `QGROUP` — the Btrfs quota group the
  space-aware cleanup measures. Empty
  until `setup-quota` has run (chapter 7).
- `SPACE_LIMIT` — the fraction (or size)
  of the filesystem snapshots may use:
  `0.5` by default.
- `FREE_LIMIT` — the fraction that must
  stay free: `0.2` by default.

**Who may look**

- `ALLOW_USERS`, `ALLOW_GROUPS` — non-root
  users who may use this config through
  the daemon. Empty means root only.
- `SYNC_ACL` — whether those permissions
  are mirrored onto the `.snapshots`
  folder itself, so named users can also
  browse snapshots as plain folders.

**Number cleanup**

- `NUMBER_CLEANUP` — on by default.
- `NUMBER_MIN_AGE` — no snapshot younger
  than this many seconds is counted or
  deleted: `1800` (half an hour).
- `NUMBER_LIMIT` — keep at most this many
  ordinary snapshots: `50`.
- `NUMBER_LIMIT_IMPORTANT` — a separate,
  smaller allowance for snapshots whose
  userdata marks them `important=yes`:
  `10`.

**Timeline cleanup**

- `TIMELINE_CREATE` — the hourly timer
  makes snapshots for this config: on by
  default.
- `TIMELINE_CLEANUP` — on by default.
- `TIMELINE_MIN_AGE` — `1800`, same idea
  as the number rule's.
- `TIMELINE_LIMIT_HOURLY` `10`,
  `TIMELINE_LIMIT_DAILY` `10`,
  `TIMELINE_LIMIT_WEEKLY` `0`,
  `TIMELINE_LIMIT_MONTHLY` `10`,
  `TIMELINE_LIMIT_YEARLY` `10` — chapter 7
  explains how the thinning works.

**Pairs**

- `EMPTY_PRE_POST_CLEANUP` and
  `EMPTY_PRE_POST_MIN_AGE` — delete
  pre/post pairs (chapter 7) that turn out
  to bracket no change at all.
- `BACKGROUND_COMPARISON` — after a post
  snapshot, pre-compute its comparison in
  the background, so a later `status` is
  instant.

## Changing the file

Edit it with `set-config`, not a text
editor, and snapper will keep the format
honest:

```
sudo snapper -c root set-config NUMBER_LIMIT=20 NUMBER_LIMIT_IMPORTANT=5
```

The lab ran exactly this shape of command
against its test config (limits 50→5 and
10→3) and then read the file: the two
lines, and only those lines, had changed.
To see the result, `get-config` again.

## Deleting a config

```
sudo snapper -c data delete-config
```

This removes the config's registration
and file **and deletes all its snapshots**
— the `.snapshots` subvolume goes with
it. It is the one snapper command in this
guide that destroys history wholesale;
read the name twice before running it.

## The check

`snapper -c root get-config` should show
your `NUMBER_LIMIT`,
`TIMELINE_LIMIT_HOURLY`, and
`TIMELINE_LIMIT_DAILY` values, and you
should be able to say what each one
caps. If `list-configs` shows nothing at
all, you already know the first place to
look: the registry.
