# Glossary

**Btrfs** — the copy-on-write filesystem
snapper is built on. Its subvolume
snapshots are the raw mechanism under
everything in this guide.

**cleanup algorithm** — a named rule that
may delete the snapshots carrying its
name: `number`, `timeline`, or
`empty-pre-post`. A snapshot with no
algorithm named is never deleted by
snapper.

**config** — snapper's unit of work: one
subvolume plus the rules for snapshotting
it, stored as a file in
`/etc/snapper/configs/` and registered in
`SNAPPER_CONFIGS`.

**copy-on-write (CoW)** — the filesystem
habit of writing changed data to new
space instead of overwriting, leaving the
old version intact for whoever still
references it. It is what makes snapshots
cheap.

**D-Bus** — the system message bus. The
`snapper` command normally asks the
`snapperd` daemon for work through it;
`--no-dbus` does the work directly.

**default subvolume** — the subvolume a
Btrfs filesystem mounts when no other is
named — in practice, the one the machine
boots. Rollback works by changing it.

**info.xml** — the small file beside each
snapshot (`/.snapshots/<n>/info.xml`)
holding its type, date, description,
cleanup algorithm, and userdata.

**post snapshot** — the second half of a
pre/post pair, taken just after a change
and linked to its pre by number.

**pre snapshot** — the first half of a
pair, taken just before a change.

**qgroup** — a Btrfs quota group: the
accounting unit snapper's `setup-quota`
creates so snapshot space can be measured
and limited.

**rollback** — making a read-write copy
of a snapshot the default subvolume and
rebooting into it, so the system runs the
snapshot's state.

**single snapshot** — a snapshot with no
partner: hand-made, timeline, and boot
snapshots are all single.

**.snapshots** — the hidden subvolume
inside each snapshotted subvolume where
its snapshots live, numbered from 1.
Number 0 in snapper's lists is not a
snapshot but the live system.

**snapperd** — the daemon that does
snapper's privileged work when asked over
D-Bus, and enforces `ALLOW_USERS` and
`ALLOW_GROUPS`.

**SNAPPER_CONFIGS** — the registry
variable in `/etc/default/snapper` listing
which config files are live. A config
file absent from it is invisible to
snapper.

**status** — snapper's comparison command:
the list of files that differ between two
snapshots (or a snapshot and now, written
`n..0`).

**subvolume** — a folder tree inside a
Btrfs filesystem that the filesystem
treats as a unit: separately mountable,
separately snapshotable. Its root always
has inode 256.

**timeline** — snapper's schedule of
hourly snapshots, and the cleanup
algorithm that thins them by age: so many
hourly, daily, weekly, monthly, yearly
kept.

**undochange** — the command that reverts,
in the live system, the file changes
between two snapshots. Files only; the
manual warns consistency is not
guaranteed.

**userdata** — key=value pairs attached
to a snapshot, beyond its description.
The pair `important=yes` has a defined
meaning to the number cleanup algorithm.
