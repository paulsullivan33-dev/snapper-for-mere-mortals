# Recipes

Short, complete jobs. Each assumes a
working `root` config from chapter 4.
Procedures that operate on snapshots are
documented-only in the same sense as
their home chapters — the commands are
snapper's documented ones.

## Snapshot before a risky edit

You are about to hand-edit something the
system depends on. Give yourself a named
way back first:

```
sudo snapper -c root create --description "before editing sshd config"
```

No cleanup algorithm — this one is yours
to keep until you delete it. Edit. Test.
If it broke, the file you need is in
`/.snapshots/<number>/snapshot/`, and
chapter 8's copy-out method is the calm
way home.

## Bracket your own change with a pair

The apt hook's trick works for any
command — `create` can run it for you:

```
sudo snapper -c root create --command "systemctl restart myservice"
```

With `--command`, snapper takes a pre
snapshot, runs the command, and takes the
matching post. Or by hand, around a
longer session:

```
sudo snapper -c root create -t pre -p -d "firewall rework"
# (note the number it prints; do the work)
sudo snapper -c root create -t post --pre-number <n> -d "firewall rework"
```

Afterwards, `snapper -c root status <pre>..<post>`
is the complete footprint of the session.

## Keep one snapshot forever, on purpose

Snapshots with no cleanup algorithm are
already safe from snapper. Make the
intention explicit and greppable:

```
sudo snapper -c root modify --userdata important=yes --description "known good, before the big upgrade" 12
```

If the snapshot *does* carry an algorithm,
`important=yes` also moves it to the
number algorithm's important allowance,
which is smaller and therefore safer.

## Thin a config that grew too fat

The list says 200 snapshots and the disk
is tight. Work in this order:

```
snapper -c root list
sudo snapper -c root cleanup all
snapper -c root list
```

Cleanup first — it may solve it. Then
look at what remains: the fat is usually
old no-algorithm snapshots (delete the
ones whose descriptions no longer mean
anything) or one ancient snapshot
pinning gigabytes of deleted data. The
used-space column names the culprits;
`delete` the oldest offenders and watch
`df` answer.

If this keeps happening, the durable fix
is in chapter 6: smaller `NUMBER_LIMIT`,
timeline limits that match how far back
you actually regret, and `setup-quota`
with ranged limits so cleanup answers to
space, not just count.

## Rehearse the rollback before you need it

On paper, in a calm moment:

1. Run `snapper -c root list` and choose
   the snapshot you would roll back to if
   tonight's upgrade broke the machine.
   Write the number down.
2. Run `snapper -c root status <n>..0`.
   Everything listed is what the rollback
   would discard. Read the list until it
   stops being surprising.
3. Check chapter 12's precondition: the
   system boots from the **default
   subvolume** (no hard-coded subvolume in
   the bootloader or fstab overriding it).
4. Note the way back from a rollback: the
   state you fled is kept as a snapshot,
   and rolling back to *it* undoes the
   rollback.

A rollback you have rehearsed on paper
is a procedure. One you have not is a
research project conducted at the worst
possible time.

## Answer "what changed last night?"

```
snapper -c root list
```

Find the day's first timeline snapshot
`A` and the current state, then:

```
snapper -c root status <A>..0
```

Filter out the noise the filters missed
by piping through `grep -v` for your
usual cache and log paths. What remains
is the day's real change list — updates,
edits, installs — in one screen.
