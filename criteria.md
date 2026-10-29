# Writing criteria for this guide

Every "for Mere Mortals" guide is written to the same nine-point
standard, so a reader who trusts one can trust the rest. Here it
is, with the verification notes for this particular guide below.

1. **Fix it first.** Reader outcomes lead, not action items: each
   chapter opens by naming the outcome it delivers, shows a check
   the reader can run to confirm they got it, and a reasonable
   reader finishes able to state what was done, what it means,
   and what to try next.
2. **Plain language.** Short sentences, everyday vocabulary,
   technical terms defined on first use. No slang, and no
   idioms that only make sense in one country — the reader may
   be translating as they go.
3. **No dead ends.** A guide never tells the reader to do
   something it cannot show them how to verify. Every claim a
   reader relies on — an install, a command, an output — is
   something this guide demonstrates or clearly labels
   documented-only below.
4. **Live-verified where it matters.** Procedures are run for
   real in the guide's lab wherever the build machine allows it,
   and the transcripts kept in the repository are the evidence.
   What could not be run is listed honestly below, never
   presented as if it had been.
5. **General audiences.** Nothing in this guide — examples,
   fixtures, names, addresses — is modelled on any particular
   reader or author's real life, network, or organisations.
   The lab's fixtures are a scratch subvolume path and stock
   descriptions.
6. **Standalone.** A reader needs this guide, its lab, and
   nothing else. Where another guide would genuinely help, it
   is pointed to once, by name, and the chapter still completes
   without it.
7. **Honest about limits.** Chapter 12 names what the tool
   cannot do: a snapshot is not a backup, there is no
   consistency promise for running programs, rollback needs a
   boot layout that honours the default subvolume, snapshots
   cost the space of everything changed since they were taken,
   and undochange restores files rather than meaning.
8. **The ethics rule is in the text, early.** Snapper raises a
   privacy point of its own kind: snapshots quietly preserve
   copies of files their owner believes deleted, readable by
   whoever holds root or an ALLOW grant. Chapter 1 and the
   README draw the adjacent line plainly — a snapshot is
   protection against change, not a backup — and chapter 2
   states who can read the `.snapshots` folder before any
   procedure depends on it.
9. **Builds clean.** The site builds with MkDocs in strict mode
   with no warnings, and a fresh reader following the chapters
   in order reaches the stated outcome of each.

## Verification notes for this guide

The build machine is an Ubuntu 24.04 sandbox (kernel 7.0.0,
x86-64), shared with other work. Snapper **0.10.6** and
btrfs-progs **6.6.3** were taken from the Ubuntu 24.04 archives
unmodified. The sandbox's apt frontend lock was held by another
process for the whole session, so the packages were fetched
with `apt-get download` and unpacked with `dpkg-deb -x`, and the
binaries were run from the unpacked tree with the library path
pointed at it. Same files an `apt install` would lay down; the
package list is in
`snapper-lab/transcripts/t01-version-and-flags.txt`.

**The mount question, bluntly.** The lab could not mount a
Btrfs filesystem, and therefore never took a snapshot. This was
not for want of trying, and transcript
`snapper-lab/transcripts/t00-environment-and-mount.txt` keeps
the whole investigation: the kernel knows Btrfs and has loop
devices (`/sys/block/loop0`–`loop7` exist), the session runs as
root with CAP_SYS_ADMIN, and a `tmpfs` mount succeeds — but
`/dev` is a curated set containing **no block devices at all**,
creating a device node with `mknod` is refused, and mounting
`devtmpfs` (which would populate the nodes) is refused. An
image file can be created and formatted but never attached.
Without a mount there is no subvolume, and snapper's snapshot
operations stop at exactly that boundary — the lab has the
error messages to prove where the boundary is.

Run for real, with transcripts as evidence:

- Version and build flags: snapper 0.10.6, libsnapper 7.2.0,
  flags `btrfs,lvm,no-ext4,xattrs,rollback,btrfs-quota,no-selinux`;
  btrfs-progs 6.6.3 (t01).
- The full `snapper --help` command map, all fifteen commands
  with their options (t02).
- A real Btrfs filesystem: a 512 MiB image created, formatted
  with `mkfs.btrfs -L labfs`, listed by `btrfs filesystem show`,
  and passed by the read-only `btrfs check`, exit 0 (t03).
- The config layer against a fake target root
  (`snapper --no-dbus --root`): `list-configs` empty, then
  still empty with a hand-written config file present;
  the same config listed once its name was added to
  `SNAPPER_CONFIGS` — proving the registry is what makes a
  config live. `get-config` read the hand-written config
  (built from the shipped default template) and printed all
  23 keys; `set-config` changed `NUMBER_LIMIT` 50→5 and
  `NUMBER_LIMIT_IMPORTANT` 10→3, confirmed in the file
  itself (t04).
- The refusal boundary, verbatim: `create-config` on a
  non-subvolume path exits 1 with `Invalid subvolume.`;
  `list` against the valid registered config fails with
  `IO error (query default id failed, subvolume is not a
  btrfs subvolume).`; `create` fails with
  `IO error (subvolume is not a btrfs subvolume).` (t04).
- The shipped machinery, read in full and quoted in the
  chapters: the default config template (every key), the
  global defaults file, the three systemd timers and their
  services — including the boot service's literal command —
  and the apt hook `/etc/apt/apt.conf.d/80snapper`, whose
  pre/post logic, `root`-config precondition, `/var/tmp`
  marker file, `DISABLE_APT_SNAPSHOT` switch, and error
  swallowing are all described from the file itself (t05);
  the complete default template is kept as t06.
- The shipped manuals: `snapper(8)` and `snapper-configs(5)`
  were read as the authority for every documented-only
  procedure, including their warnings about database
  consistency and undochange, which the guide quotes rather
  than paraphrases away.

Documented only — described from the project's shipped
manuals, not run, and labelled as such where they appear:

- **Every snapshot operation**: creating single, pre, and post
  snapshots; listing snapshots on a real system; deleting;
  `setup-quota`; and all three cleanup algorithms running
  against real excess snapshots. The algorithms, limits, and
  defaults in chapter 7 are the manual's and the template's,
  both read in full.
- **Comparison and recovery**: `status`, `diff`, and `xadiff`
  output; copying a file out of a snapshot; `undochange`;
  and `rollback` in both its forms — including the default
  subvolume mechanism, which the lab could not touch.
- **The automation firing**: the timeline, cleanup, and boot
  timers' units were read, but no timer ever fired against a
  live config; the apt hook's bracketing of a real install is
  likewise described from its file, not observed.
- The neighbour tools of chapter 11 (hand-rolled Btrfs
  snapshots, `btrfs send`/`receive`, Timeshift, backup tools)
  characterised from their documentation, not run.

Fixtures are this guide's own and deliberately generic: a
scratch path named `/data`, stock descriptions such as "first
snapshot" and "before editing sshd config", and no names,
organisations, or real events.
