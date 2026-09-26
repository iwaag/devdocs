# sage p2 — step 2 report: study setup and its return path

## What changed

- **autolab lays a study out from its setup request** (agautolab `9d52afd`,
  `15853c2`, module `study_setup`). Before `init_project`, a serving of a
  project channel with no `README_PROJECT.md` looks for the `ag-setup`
  block (`ag.project-setup.v1`, written by `agproject`):
  - It first checks the history the serving already has.
  - It reads `workplan-setup-<slug>` only when the workspace does not exist
    yet. That way an older project without a marker costs no extra read per
    serving.

  For `pattern: study`, the steps are:
  1. Create `autodev/<slug>`, clone it into `main/` and add the `.local/`
     ignore.
  2. Write `RESEARCHPLAN.md`: the posted plan by message id, with
     `agproject`'s origin line stripped.
  3. Write `README.md`, `methods/README.md` and `reports/INDEX.md` (one
     row per investigation).
  4. Commit and push.
  5. Only then write `README_PROJECT.md`, generated from the block.

  A failure half way leaves no marker and the next serving retries the whole
  sequence. `init_project` then sees the marker, so no `direction/` or
  `devlog/` repository is ever made. Existing files are never overwritten.
  The planner still reads the request and may refine `main/README.md`, and
  its edit is committed as planning notes.
- **The answer says so in one machine-readable line**:
  `study layout established: main/ = <repository> at <commit>`. The commit
  is `main`'s HEAD when the answer is sent. The line is owed once in the
  setup topic, again after a serving that established the study but
  crashed before answering, and never twice.
- **The marker can be rebuilt from its record.** The block lives in Zulip,
  not on the disk. `autolab project establish <slug>
  [--rewrite-marker]` rebuilds `README_PROJECT.md` from it.
- **`agproject status` counts only real answers** (pyagag `a73c216`,
  found live, see below):
  - an acknowledgement is not an answer;
  - a structured setup is `ready` only by the established line, since the
    repository exists before the planner has read the request;
  - an older prose setup can still be `ready` by its repository.
- **archsage is called back where it delegated** (archsage `0ef6c2c`):
  - A mention outside an argue that carries archsage's own root note serves
    the conversation that note names. That conversation is located by its
    anchor, and a ✔'d home is served under its ✔ name, so no twin is
    opened.
  - The calling topic is placed beside the chatlog as a thread
    (`threads/`, and summarized in the prompt), and the reply goes home.
  - The served mark is written by the executor after delivery.
  - A mention with no root note of ours is still left alone. An argue
    remains invitation-only.
- **Replies in archsage's own channel name the requester**, so an agent
  that asked (Front delegating a study) is called back. A reply that
  declares `intent=progress` names nobody (`TopicResult.quiet_progress`,
  pyagag), because waiting should buy nobody a run.
- **Ownership and the return destination.** Study establishment is owned
  by a conversation in **archsage's own channel** (`study-<slug>` by
  convention). An argue or a Front Desk conversation that wants a study
  asks there, through `agentchat send`, which writes the asker's root
  note. The requesting conversation is therefore preserved as the asker's
  home, and archsage's final report names the asker, whose own callback
  route brings it home. The alternative was for archsage to continue
  inside the argue. That was rejected: in an argue archsage speaks only
  when invited, and it has no home there to be called back into.
- archsage now holds `agproject`, `agroutine` and `agentchat`.
  `AGAG_PROVISIONER_ENV` comes from the ignored `.local/listener.env`,
  which `service/listen.sh` sources.

## Verification

- **Tests:**
  - autolab `tests/test_study_setup.py` (5) covers:
    - an empty workspace becoming a study, with no ordinary scaffold and a
      second serving that changes nothing;
    - a failed push leaving no marker, then a clean retry;
    - finding the block from another topic;
    - another pattern left alone;
    - the established line said once, again after a crash, and never
      twice;
    - the marker rebuilt from the block.
  - archsage `tests/test_listener.py`:
    - a callback serves home with the answer as a thread and replies to
      home's requester, with the served mark;
    - a ✔'d home is served under its ✔ name;
    - a progress reply names nobody.
  - Full suites: pyagag 919, agautolab 304, archsage 31 and agfront 179
    passed.
- **Live probe 1, the normal callback** (`#archsage-agstudio1 ›
  study-sagep2probe`, from #11495):
  - archsage ran `agproject open` and wrote plan #11497, the root note
    #11498 and setup #11499 with the block.
  - autolab laid out the study (`README_PROJECT.md` and `main/` only),
    answered #11502 with the established line, and named archsage.
  - archsage was called back and served home (log: `mention in
    'pj-sagep2probe'/'workplan-setup-sagep2probe' serves
    archsage-agstudio1/study-sagep2probe #11495`).
  - It reported to the Omni Agent (#11504) and wrote served mark #11505.
  - **Failure found:** archsage's first reply (#11501) already said
    `ready`. `status` had counted autolab's "Message received" ack
    (#11500) as an answer, and the repository already existed. This was
    fixed in pyagag `a73c216` (above) before probe 2.
  - The established line named the layout commit rather than the
    planner's later README commit. This was fixed in autolab `15853c2`.
- **Live probe 2, restart while pending and a callback after
  resolution** (`study-sagep2probe2`, from #11506):
  1. autolab's listener was stopped. archsage opened the study, saw
     `setup-pending`, and ended with #11511, a progress reply naming
     nobody.
  2. archsage's listener was stopped and autolab restarted. autolab
     answered #11513 (`… at 659f5fbca107`) while archsage was down.
  3. The Omni Agent ✔'d the setup topic.
  4. On restart, archsage's startup recovery found #11513 in the ✔'d topic
     ("names us and has no receipt"), served
     `study-sagep2probe2 #11506`, and reported `ready` with the revision to
     the Omni Agent (#11516). The served mark is #11517.
- **Cleanup:** both probes were archived with `agautolab.project_archive`:
  - the Gitea repositories are archived;
  - `-direction`/`-devlog` are `absent`, which confirms no ordinary
    scaffold;
  - the workspaces were moved aside, and the channels and folders were
    archived;
  - the archsage topics are ✔.

## Notes

- archsage's callback report began with the listener's `@**Omni Agent**`
  and repeated it in its own words. The guide will say that the listener
  names the requester (step 4).
- The probes gave archsage explicit steps. The workflow without step-by-step
  instructions is exercised in step 5.
