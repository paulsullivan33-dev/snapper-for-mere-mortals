# The Moving Parts

By the end of this chapter you will know
which piece of snapper holds what: the
command, the daemon, the config files,
the `.snapshots` folder, and the timers.
The check: for each of the five, say
where it lives on disk. Every path in
this chapter comes from the package the
lab unpacked and read — this chapter is
verified, not documented-only.

## The command: `snapper`

`/usr/bin/snapper` is the tool you type.
It lists configs and snapshots, creates,
modifies, and deletes them, compares two
snapshots, undoes changes, and rolls
back. The full command map is long —
fifteen commands — and the lab kept it as
a transcript (`snapper-lab/transcripts/t02-full-help.txt`).
The shape to remember: almost every
command works on **one config**, chosen
with `-c`, and there are global switches
for machine-readable output (`--csvout`,
`--jsonout`) that make snapper friendly
to scripts.

Run `snapper --version` and you learn
more than the version. The lab's build
answered:

```
snapper 0.10.6
libsnapper 7.2.0
flags btrfs,lvm,no-ext4,xattrs,rollback,btrfs-quota,no-selinux
```

Those flags are the build's honest
feature list: Btrfs support in, LVM thin
snapshots in, ext4 support **out** (it was
discontinued upstream), extended
attributes in, rollback in, Btrfs quota
support in. Chapter 3 makes a check of
them, because a guide that tells you to
roll back on a build compiled without
rollback would be a trap.

## The daemon: `snapperd`

Most of the time you never talk to
snapper directly. The `snapper` command
asks a daemon, **snapperd**, to do the
work over **D-Bus**, the system message
bus. The daemon runs as root (snapshots
are privileged work), is started on
demand by D-Bus — its service file is
`org.opensuse.Snapper.service` — and
decides who may ask: by default root, plus
any users and groups a config names in
its `ALLOW_USERS` and `ALLOW_GROUPS`
settings.

Two consequences you will meet later.
First, `snapper --no-dbus` exists: the
command does the work itself, which must
be run as root, and is the mode installers
and rescue systems use. Second,
`snapper --root <path>` operates on
another system's root folder — and only
works together with `--no-dbus`. The lab
used exactly that pair to exercise the
config layer against a test root.

## The configs: `/etc/snapper/configs/`

A **config** is snapper's unit of work:
one subvolume to snapshot, plus all the
rules for how — how many snapshots to
keep, whether the timeline runs, who may
look. Each config is one plain text file
in `/etc/snapper/configs/`, named after
the config. The root filesystem's config
is conventionally named `root`.

A config file only counts when it is also
**registered**: the list of live configs
is the `SNAPPER_CONFIGS` variable in
`/etc/default/snapper` (on SUSE systems,
`/etc/sysconfig/snapper`). The lab proved
this the plain way: it placed a valid
config file in a test root and
`snapper list-configs` showed nothing;
after adding the name to `SNAPPER_CONFIGS`,
the same command listed it. `create-config`
writes the file and registers it in one
step; chapter 6 covers the file's contents
setting by setting.

Templates for new configs live in
`/usr/share/snapper/config-templates/`
(the lab read the `default` template in
full — every new config starts as a copy
of it, with the subvolume filled in).

## The snapshots: `<subvolume>/.snapshots/`

Snapshots live inside the subvolume they
copy, in a hidden subvolume named
`.snapshots`. Snapshot number 7 of the
root subvolume is the folder tree
`/.snapshots/7/snapshot` — itself a real,
read-only Btrfs snapshot — with a small
`info.xml` file beside it at
`/.snapshots/7/info.xml` holding the
bookkeeping: the type, date, description,
cleanup algorithm, and userdata. The
numbers are never reused while a snapshot
lives, and in lists, number **0** is a
stand-in for the live system itself —
"compare snapshot 4 with now" is written
`4..0`.

Two more residents of the package are
worth knowing. The **filters** in
`/usr/share/snapper/filters/` list paths
excluded from comparisons, so `status`
does not drown you in cache and log churn.
And everything snapperd does is logged to
`/var/log/snapper.log` — the file the
troubleshooting chapter sends you to.

## The timers: three systemd units

The package ships three timers, and the
lab read all three:

- **snapper-timeline.timer** fires hourly
  (`OnCalendar=hourly`). Its service runs
  snapper's systemd helper, which creates
  a timeline snapshot for every config
  with `TIMELINE_CREATE=yes`, then runs
  the cleanups that are due.
- **snapper-cleanup.timer** fires daily,
  starting ten minutes after boot. Its
  service runs the cleanup helper across
  all configs — the number, timeline, and
  empty-pair algorithms of chapter 7.
- **snapper-boot.timer** fires a second
  after boot. Its service is a single
  visible command:
  `snapper --config root create --cleanup-algorithm number --description "boot"`.

Notice the pattern: snapper's automation
is not magic inside the daemon. It is
timers running the same command you can
run, driven by the same config files.
When automation misbehaves, there is
always a command to run by hand and watch.

## The check

Without scrolling: the daemon is named
**snapperd**, the registry variable is
**SNAPPER_CONFIGS**, snapshots live in
**`.snapshots`** inside their subvolume,
and the log is **`/var/log/snapper.log`**.
