# workflow_editor fix1 — Step 3: the common guide

agautolab `69142b8`, `agent/guides/wfrun/guide.md`, *Shared data*.

## What the guide now says

The paragraph used to say the share is "named `shared://<share>/<path>` in
the person's words". It now states, as facts of the agent's world:

- the share (e.g. `agshare`, an NFS export) is where people put materials
  and handoff folders for agents and pick results up — the first place to
  look for what a person supplies;
- people describe items in their own words (`/agshare/…`, `agshare/…`, a
  share-relative path, "the folder on the NFS", another PC's mount path),
  neither as references nor as paths of the host the agent runs on; an
  execution host reaches a share at its own folder, which may be the
  server's backing folder rather than a mount — with the run-001 failure
  as the reference;
- `wfe share resolve` turns such a path into a reference, `wfe share find`
  searches by name with bounds; an incomplete search or an unreachable
  share is not absence;
- what remains ambiguous after looking goes to the person, and the
  commands are the agent's, never the person's;
- a path that plainly names the project's own files stays the project's;
- records keep the reference.

No command sequence is prescribed; details stay in `wfe share --help` and
`docs/shares.md`.

## One place

The fact lives once, in the wfrun guide that every run (including Front's
conversations and every project's workflows) reads. Project guides that
mention shares (lowerpoly, playprey, Front's glossary) use `shared://` as
the name of a known location in records, which stays correct, so they were
not changed. The guide is read at each run's start from this checkout; no
restart is needed.
