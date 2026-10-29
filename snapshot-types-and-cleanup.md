# Snapshot Types and Cleanup

By the end of this chapter you will know
the three snapshot types, how each one
gets its cleanup algorithm, and how the
three algorithms decide what dies. The
check: look at a snapshot list and, for
each row, say which algorithm owns it —
or that none does, and it lives until you
delete it by hand.

> **Lab note.** Creating and deleting
> snapshots needs a mounted Btrfs
> filesystem, so this chapter is
> **documented-only**: it is written from
> snapper's shipped manual and the default
> config template, both of which the lab
> read in full. The defaults quoted are
> the template's actual numbers.

## Three types, one mechanism

Filesystem-wise, every snapshot is the
same read-only Btrfs snapshot. The types
are bookkeeping — a promise about why the
snapshot exists:

- **single** — a snapshot on its own. What
  `create` makes by default, what the
  timeline makes, what the boot timer
  makes.
- **pre** — the first half of a pair,
  taken just *before* a change.
- **post** — the second half, taken just
  *after*, linked to its pre by number.

The pair is the unit that answers "what
did that operation change?" — `status`
between a pre and its post is the
operation's footprint, and the
empty-pre-post cleanup exists to drop
pairs whose footprint is zero.

## The cleanup column is the lifespan

When a snapshot is created, it can carry
a **cleanup algorithm** name — by hand
with `--cleanup-algorithm`, or
automatically by whoever created it. The
name appears in the list's cleanup
column. It decides which algorithm may
delete the snapshot later.

A snapshot with an **empty** cleanup
column belongs to no algorithm. Snapper
will never delete it. That is the right
default for snapshots you take by hand
before something risky — and a slow leak
if everything you create looks like that.

## Algorithm one: number

The simplest rule: keep at most N.
Snapshots carrying the `number` algorithm
are counted newest-first; those past the
limit are deleted, oldest first, except
that none younger than `NUMBER_MIN_AGE`
(1800 seconds) is touched. Two counters
run side by side: ordinary snapshots
against `NUMBER_LIMIT` (50), and
snapshots marked `important=yes` in their
userdata against `NUMBER_LIMIT_IMPORTANT`
(10). Marking a snapshot important is how
you say "this one of the fifty matters
more":

```
sudo snapper -c root modify --userdata important=yes 7
```

Who feeds this algorithm? The boot timer
(its snapshots are `number`), the apt
hook of chapter 9, and on openSUSE the
package tools. Fifty boots and package
operations is a deep bench — most home
machines can lower `NUMBER_LIMIT` a lot.

## Algorithm two: timeline

The timeline keeps **spread**, not count:
the last `TIMELINE_LIMIT_HOURLY` hours
each keep one snapshot, the last
`TIMELINE_LIMIT_DAILY` days keep one, and
so on for weeks, months, years. Defaults
keep 10 hourly, 10 daily, 0 weekly, 10
monthly, 10 yearly — a curve that is
dense near now and sparse far away,
which matches how regret works: you
almost always want the recent past, and
the distant past only coarsely. Nothing
younger than `TIMELINE_MIN_AGE` is
thinned.

The hourly snapshots the timer makes
carry the `timeline` algorithm. So does a
snapshot you create with
`--cleanup-algorithm timeline`. Run the
thinning by hand any time with
`snapper -c root cleanup timeline`.

## Algorithm three: empty-pre-post

A pre/post pair that brackets a change
that changed nothing — a package
operation that failed before touching a
file, say — is pure clutter. This
algorithm deletes such pairs once they
are older than `EMPTY_PRE_POST_MIN_AGE`
(1800 seconds). It needs the pair's
comparison; `BACKGROUND_COMPARISON=yes`
(the default) makes sure that comparison
is already computed.

## Running cleanup, and the space-aware pass

```
sudo snapper -c root cleanup all
```

runs every algorithm for the config —
this is what the daily cleanup timer
does. `cleanup number`, `cleanup
timeline`, and `cleanup empty-pre-post`
run one each. And `snapper cleanup --path /`
cleans every config touching a path.

There is a second gear. If quota is set
up —

```
sudo snapper -c root setup-quota
```

— and a limit is written as a **range**,
like `NUMBER_LIMIT="2-50"`, the algorithm
works in two passes: first delete down to
the range's maximum no matter what, then
keep deleting down toward the minimum
until snapshots use less than
`SPACE_LIMIT` (half the filesystem) and
more than `FREE_LIMIT` (a fifth) stays
free. Ranges turn cleanup from a counting
rule into a space guarantee. The quota
setup is what fills the config's `QGROUP`
key and lets snapper measure how much
space each snapshot truly owns — the same
measurement behind the list's used-space
column.

## Deleting by hand

```
sudo snapper -c root delete 12
sudo snapper -c root delete 12-18
```

Ranges and lists both work. Deleting
frees exactly the blocks no other
snapshot references — deleting a snapshot
whose neighbours bracket the same changes
frees little; deleting the oldest, with a
long tail of changes only it remembers,
frees the most.

## The check

Pick three rows from
`snapper -c root list`. For each: its
type, its cleanup column, and therefore
its fate — counted by number, thinned by
timeline, dropped as an empty pair, or
kept forever until you say otherwise.
