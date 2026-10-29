# Snapper and Package Managers

By the end of this chapter you will know
who creates snapshots when you are not
looking — the apt hook, the timeline
timer, and the boot timer — and how to
turn each of them off. The check: install
or remove a small package, then find the
pair of snapshots the operation made.

> **Lab note.** The apt hook in this
> chapter is **verified**: the lab read
> the installed file line by line
> (transcript
> `snapper-lab/transcripts/t05-units-and-hooks.txt`),
> and the timers were read the same way.
> Watching a pair actually appear around
> an install needs a mounted Btrfs root,
> so that step is documented-only.

## The idea: bracket the change

A package operation is the change most
likely to break a system, and it has a
clean before and after. Snapper's design
brackets it: a **pre** snapshot a moment
before the package tools touch the
system, a **post** snapshot the moment
they finish. The pair shares a
description, the post knows its pre's
number, and `status pre..post` afterward
is a complete list of what the operation
changed on disk. Chapter 7's
empty-pre-post cleanup quietly deletes
pairs that bracket nothing.

On openSUSE this bracketing is built into
YaST and zypper. The Ubuntu package
achieves the same thing with an apt hook.

## The apt hook, line by line

Debian and Ubuntu run dpkg hooks named in
`/etc/apt/apt.conf.d/80snapper`, which
**every** `apt install`, `apt upgrade`,
or direct `dpkg` run passes through. The
lab read the file. Its logic, in plain
words:

**Before** any package work
(`DPkg::Pre-Invoke`): if snapper is
installed and a config file named `root`
exists at `/etc/snapper/configs/root`,
take a pre snapshot —

```
snapper create -d apt -c number -t pre -p
```

— print its number into a marker file,
`/var/tmp/snapper-apt`, and run
`snapper cleanup number` so the bench
stays trimmed.

**After** the work (`DPkg::Post-Invoke`):
if the marker file exists, take the
matching post snapshot with
`--pre-number` read from the marker, and
clean up again.

Three details worth knowing:

- **The name `root` is a switch.** No
  config named `root`, no apt snapshots.
  This is why chapter 4's convention —
  name the root filesystem's config
  `root` — is more than tidiness.
- **The master switch** is in
  `/etc/default/snapper`: set
  `DISABLE_APT_SNAPSHOT="yes"` and the
  hook skips its work.
- **Failures are silent by design.** Both
  hooks end their commands with
  `|| true`. A snapper problem can never
  block a package install — the price is
  that a pair can quietly fail to appear,
  which is a troubleshooting entry in
  chapter 13.

## The timeline timer

Independently of packages, the
`snapper-timeline.timer` unit fires
**hourly** (that is its literal
`OnCalendar=hourly`). Its service runs
snapper's systemd helper, which, for each
config with `TIMELINE_CREATE=yes`, takes
a timeline snapshot if the hour needs
one, then runs the timeline cleanup and
any other cleanups that are due. This is
the drumbeat that gives chapter 7's
timeline its raw material: on a machine
that is switched off at night, the
timeline simply has gaps, and the
algorithm keeps what exists.

The matching daily sweep is
`snapper-cleanup.timer`, starting ten
minutes after boot and then daily,
running all cleanup algorithms across
all configs — insurance for algorithms
whose snapshots are created by hand,
outside the timeline's rhythm.

## The boot timer

`snapper-boot.timer` fires one second
after boot. Its service is the plainest
of the three — the unit literally runs:

```
snapper --config root create --cleanup-algorithm number --description "boot"
```

One snapshot per boot, owned by the
number algorithm, so a machine rebooted
daily keeps a rolling bench of recent
boots without the count growing without
bound. On a machine that never reboots,
this contributes nothing — the timeline
is doing that job.

## Reading the automatics in the list

`snapper -c root list` then tells the
whole story at a glance: `apt` pairs by
description, hourly `single` snapshots
carrying the `timeline` cleanup, `boot`
snapshots carrying `number`. Chapter 7's
check — name the owner of each row — is
the skill that turns the list from a wall
of numbers into a diary of what the
machine has been through.

## The check

Run `apt install` on something small and
harmless, then `snapper -c root list`.
Find the newest **pre** and **post** rows,
note their numbers, and run
`snapper -c root status <pre>..<post>`:
that list is everything the install
changed. If no pair appeared, chapter 13
has the three usual reasons.
