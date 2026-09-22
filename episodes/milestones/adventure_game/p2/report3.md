# Step 3 report — references through planning, generation and research

Plan: [plan.md](plan.md) step 3. 2026-09-22, by the Omni Agent. The mechanism is built,
tested and deployed on this Mac; the live end-to-end check waits for the human's first
push (report 2, *Pending*) and is recorded at the top of step 4.

## Discoverability and tools

One tool for every role, `agrefs` (pyagag `b227b78`), under a `Bash(agrefs:*)` grant:

| Agent | Roles | Commit |
|---|---|---|
| Front | `front`, `desk`, `argue` | agfront `244a50c` |
| autolab | `front`, `director`, `mediator`, `coding`, `superdirector`, `supercoder`, `argue` (not `summarizer`) | agautolab `4939ca9` |
| forge | `front`, `generator`, `argue` | agforge `43730aa` |
| archsage | `archsage` (the council) and its `front`; **not** the sages | archsage `b8099ac` |

pj-agdev `34e83c4` pins the three submodules. All four listeners were restarted on the
new pin at 07:41–07:42Z and recovered 0 pending conversations. Tests: agfront 181,
agautolab 247, agforge 265 (3 new for `--init-image`), archsage 28, pyagag 694.

Each guide got the same paragraph — what a reference is, the verbs, "work from the
revision you were given, quote `<source>@<rev>:<path>`, never substitute silently,
originals are read-only, say when a reference and the request disagree" — plus a
role-specific paragraph:

- **Front** (`front`, `desk`): resolve what the developer names and carry
  `<source>@<rev>:<path>` into `GOAL.md`, `workplan-` posts and asset requests; adopt a
  commit, never `latest`. **`argue`**: name references in invitations so an agent opens
  the same file.
- **autolab `workplan_superdirector`**: `agrefs sync` first; record the adoption in
  `direction/REFERENCES.md` (source, commit, date, why, mission); each `task[N].md`
  names the paths it works from; a mission keeps its revision until a request adopts a
  new one, and `agrefs changes <old>..<new>` bounds what a change touches.
  **`workrun_supercoder`**: read originals at the named revision, look at images through
  `agrefs path`, ask forge by identity, report reused / transformed / new and departures;
  decisions in `direction/`, derivatives in `main/`.
- **forge `assetplan_front`**: write the identity into `required_items.md` with what it
  establishes. **`assetplan_generator`**: `--init-image "$(agrefs path …)"`; say which
  reference steers what and how; creative direction outranks local and general
  knowledge, the requester's words outrank both (the precedence rule the p1 hint asked
  to integrate). **`assetrun_generator`**: the report names the reference used.
- **archsage**: creative references are a project's inputs, not findings; nothing from
  them is queued into a sage's tree; cite them apart from what the trees found.

## Media

- Text and trees: `agrefs show` (text, directory listing with sizes) and `search`.
- Images: `show` on a binary answers `binary: image/png, 1920x1080, 2,347,113 bytes, at
  <path>` — the p1 hint's "a path alone does not prove visual access" is met by the
  harness's own image reader over that path (claude_code's `Read` renders images; the
  Omni Agent used the same to check the generation below).
- **Reference images into generation**: `agforge image generate --init-image PATH
  --init-creativity F` sends SwarmUI `initimage` (data URL) and `initimagecreativity`.
  Checked live: the v0.1.0 meadow still as init image at creativity 0.5, 512×288, 12
  steps, prompt "the same meadow at dusk, seen from grass height, warm low sun" → an
  image keeping the still's low sun, tree line and grass-height framing (viewed).
  `toolset-image.md` documents the flags.
- Executable examples: the snapshot is a plain directory; a task copies what it wants
  to try into its working copy (guide).

## Where things are recorded

- Interpretation, scope, decisions, adopted revision: `direction/` (`REFERENCES.md`).
- Derivatives and implementation: `main/`.
- Evidence: the ordinary run topics and forge's `assetrun-` records; departures and
  unresolved conflicts are what the guides tell each role to state.

## Live checks after the human's push (2026-09-23)

The Developer pushed `a3c8196` ("first advice") on 2026-09-23 — and reorganised the
tree as they saw fit (`human_advice.md`, `todo.md`, `preds/wolf/{images,texts}/`; the
scaffold's suggested folders and README are gone, which is exactly the freedom the
contract gives them).

| Check | Result |
|---|---|
| `agrefs sync protoprey-refs` in agfront, agautolab, agforge, archsage | each resolved `a3c8196` (`a3c81968de…`) and laid out its own snapshot |
| `agrefs show …:preds/wolf/images/catch.jpg` | `binary: image/jpeg, 1344x768, 637,975 bytes, at <path>` |
| Independent retrieval on agautolab1 (plain `git clone` through the ansible key) | HEAD `a3c8196`, tree `113e2785…` = the tree of the human's clone; 14 files; `sha256` of `catch.jpg` and `catch.txt` byte-identical to the snapshot here |
| Embedded generation parameters (the Developer asked whether agents can read them) | readable: EXIF `UserComment` (UTF-16) holds SwarmUI `sui_image_params` — prompt, negative prompt, model `oneObsession_v22`, seed, steps, cfgscale — for all five images |
| The five images viewed by the Omni Agent | discovery (low-angle wolf walking the forest path), catch (open jaws from below, tongue, teeth, saliva), carry (side view, jaws slightly open, drool), swallow (muzzle raised to the moon, eyes closed), rest (wolf asleep in a cave mouth) — the phases of the event, as the advice describes them |

The delegated-result check (a result that names its reference) is step 4's first
mission, recorded in report 4.

## Not done, on purpose

- No change to forge's `knowledge.py` (references are not media-making knowledge) and
  none to `sagetree`.
- No deploy to agautolab1 (`b38a6af`): nothing in this episode runs there; the plan's
  cross-machine check is a retrieval check, done with plain git through the ansible key.
- No manifest, no per-file metadata, no resolver service.

## Cost of the step

One SwarmUI generation (local GPU, no paid run). No agent run was bought.
