# agent_guide p1 — step 1: inventory

This inventory covers every paragraph of the conversational guides in
agfront, plus the paragraphs copied into the guides of other agents. For
each one it gives the commit that added it (`git blame` in `pj-agdev/agfront`,
with commit subjects and bodies), the evidence behind it, where else the
same text appears, and where it goes next. Later steps delete text only
where a row below says so.

## Classes

| class | meaning | destination |
|---|---|---|
| **F** fact | a fact the agent works under; no tool can say it | guide (head or body), short |
| **T** tool | describes what one command does | that command's `--help` (the guide keeps at most one line) |
| **S** shared | the same text in several roles or agents | written once (a shared file or generated section); per-role text stays in the role's guide |
| **A** anxiety | no trial or failure behind it | may be dropped; listed in `report.md` |

Paragraphs usually mix classes, e.g. **T+F**: the usage goes to `--help`
and a one-sentence fact stays in the guide. The Evidence column states
what the commit or report cites. "design" means the paragraph came with a
new feature and has no failure behind it. That is not anxiety: the
paragraph moves, but it is not dropped.

## Evidence keys

| key | what was seen |
|---|---|
| fd1 | front_desk p1 step 4, seen live: reasoning preface posted; stray report posted into `routine-<name>` |
| fd-wr | `front-desk-20260908-161951`: Front opened `workrun-task1-g-15` by hand; posted a second start into a ✔ topic (`ecf27f9`) |
| fd2 | front_desk p2 step 5: preface seen live again (`143908c`) |
| as9 | agent_standardize p9: "how is it going?" restarted forge's generation and misrouted its delivery (`df8da45`) |
| as5-8 | agent_standardize p5–p8: backgrounded wait, callback loops; runs are one reply (`35bb43a`, `586cc6f`) |
| prop | Front skipped the proposal in its first two live runs (`233f6cd`, devenv) |
| rt1 | routine_tests p1 step 3: four self-ended runs in three (`acea1fc`) |
| rt2 | routine_tests p2 ex1 B1: Front delegated for a run it had just opened; the run was lost (`53bbfb9`) |
| ag1 | argue p1 step 4: `@**cagent**` reached nobody; "I will bring in X next" invited nobody (`03d2682`) |
| agm1 | adventure_game p1 step 4 (2026-09-20, ProtoPrey): a start decided after the argue had nobody to act on it (`4538f9e`) |
| agm2 | adventure_game p1/p2: "I played it" / "confirmed it works" written twice (`8b4674a`) |
| fs1 | failsafe p1 step 3: Observer stop requests (`2583175`) |
| fs2 | failsafe p2 trials A/A2: dead process behind an open serving; resume taken as agreement (`bcba734`) |
| fs3A | failsafe p3 trial A: Front took its own ack as the work moving (`7551da6`) |
| fs3A2 | failsafe p3 trial A2: Front said "closed" before the record (`19f2bfa`) |
| fs4 | failsafe p4: Front could misread recovery evidence → `recheck` (T1–T3; T3 abstained on RESUMING) (`b64de4b`) |
| fs5 | failsafe p5 step 3/4: acceptance by the decision holder, `reserve` (trial G); `agrun` close-out (`61ffa8c`, `5bd9216`) |
| fs5E | failsafe p5 trial E: agreement posted into a topic nobody serves; recheck said ASKED (`7b42c2c`) |
| fs6r | failsafe p6 F5: hand-written receipt #15362 parsed by nobody (`3207a39`) |
| fs6h | failsafe p6 step 4: holds (design; used live in p6 report6) (`4939026`) |
| fs6x1 | failsafe p6 ex1: relations/dispositions (design; used live without further guidance, ex1 report) (`75d739f`, `93a133c`) |
| fs6x2 | failsafe p6 ex2: Front refused the Omni Agent's confirmation (ex1 L3) → proxy authority (`12ce0f1`) |
| pp1 | progress_panel p1 step 5 trial: a run completed by record stayed open (`db03c21`) |
| rp | runtime-profile / refine_routine p1: execution options, windows (design; proven live 2026-09-09) (`40c48dd`, `efc1910`, `9ac28ce`) |
| sp2 | sage p2 step 4: study establishment routed to archsage; "setup is not research" (`1419002`) |
| inc | this phase's incident, run-0160 (`preresearch.md` §1) |

## desk (`agent/guides/desk/guide.md`, 367 lines)

"Also in" uses f = front, r = routine_run, a = argue.

| # | lines | content | commit | evidence | also in | subcommand | class → destination |
|---|---|---|---|---|---|---|---|
| D1 | 1–2 | You are Front at the Front Desk; the reply goes to the developer | 1be5b8f, f7508c0 | design | f (1 line) | — | F → head |
| D2 | 4–8 | reply in the developer's language, exact names; this conversation is the record; no role-play/dialogue | 03bf127 | design (argue p2 removed the character) | — | — | F → head (desk only) |
| D3 | 10–12 | posts to other agents are ordinary professional language | 1be5b8f | none: written for the in-character voice, which is gone | r (step 5) | send | A → one clause in shared text |
| D4 | 14–21 | a speaker marked `— with <person>'s full authority` is that person's decision; record it with its own post | 12ce0f1 | fs6x2 | f | accept/hold/release/disposition | F+S → shared head |
| D5 | 23–25 | no `@**mentions**` in `#front` (public channel; a mention summons) | 1be5b8f | mechanism (MEMORY: #front public) | — (f lacks it) | — | F+S → shared head |
| D6 | 29 | read `chatlog.md` and the threads beside it | 1be5b8f | — | f, r | — | F → head ("what you have") |
| D7 | 31–32 | small talk or plain-text question → just reply | 1be5b8f | **against it**: inc (run-0160 took this branch) | f (l.4) | — | A → drop; greetings only if at all |
| D8 | 33–36 | work asked → read `tools/`, propose (agent, where), ask; unless told to proceed | 1be5b8f | prop | f (l.5–7) | — | F+S → shared body as a fact ("the developer approves a delegation unless…") |
| D9 | 37–39 | plan accepted → talk via agentchat; report channel/topic and what you asked | 1be5b8f | as5-8 | f (l.9) | send | F+S → merge with D42 |
| D10 | 40 | nothing can do it → say so kindly | 1be5b8f | none | f (l.7) | — | A → drop |
| D11 | 41 | already done → say so | 1be5b8f | none | f (l.209) | — | A → drop |
| D12 | 43–55 | argue: what it is; `argue open <stem>`; its own conversation; do not post again or delegate for it | d07d742 | design; anchoring mechanism = rt2 | f (l.11–24) | argue open, topics | T+F+S → `argue open --help` carries the usage; the fact (its own conversation, served there) is shared |
| D13 | 57–63 | developer decided a project → write `GOAL.md`, `agproject open …`; no developer step; nothing starts | 1419002, 4538f9e | agm1, sp2 | f (longer: hand first work as `workplan-`, "a plan of yours is not a decision"), a | agproject open | T+F+S → GOAL.md contents to `agproject open --help`; fact (you open it, only on a stated decision) shared |
| D14 | 65–78 | a study is archsage's to establish; what to send; setup ≠ research; refresh the sage after the run | 1419002 | sp2 | f, a (l.119–124) | — | F+S; the routing part repeats archsage's introduction (`params/intro.md` §Establishing a study) → one line "archsage establishes studies; its introduction says how"; keep "setup complete ≠ research done" |
| D15 | 80–83 | routine = guide in Zulip; folder `routine`; newest post in `guide` is the guide; never post into `guide` | 679d8fe, 120a872 | fd1 (stray post into routine channel) | f, r (l.77) | channels, read | F+S; the discovery command goes to the head |
| D16 | 85–96 | open a run: one post into `routinerun-<id>`, opening post contents; do not post again; report comes here; no schedule | cbafb8a, 679d8fe | design (refine_routine) | f (+ execution preference, rp) | send | T+F+S → "how a run is opened" goes to `agrun --help` (agrun owns runs); fact shared |
| D17 | 97, 138–141 | opening the run is the whole work; move stray work under it with `agrun adopt` | cbafb8a, 5bd9216 | rt2 | f (l.74–84, longer, with the mechanism) | agrun adopt | F+S (keep rt2's mechanism sentence) |
| D18 | 98–113 | who accepts a mission: entrusted → you; reserved → the developer's words; initial request ≠ acceptance; do not post acceptance into `workplan-` | 61ffa8c | fs5 | f, r (l.68–75) | accept, reserve | T+F+S → the mechanics are already in `accept`/`reserve --help`; the fact of who holds the decision stays, shared |
| D19 | 115–120 | a run's last steps may happen here; `tools/runs.md`; `agrun status` | 5bd9216 | fs5 | f | agrun status | T → `agrun --help`; F: "an open run is yours" |
| D20 | 122–127 | `agrun continue … --because`; your post in the run topic serves nothing | 5bd9216 | fs5 | f | agrun continue | T → `agrun continue --help` (with the "a post serves nothing" fact) |
| D21 | 128–136 | `agrun finish`; a sentence saying complete ends nothing | db03c21 | pp1 | f | agrun finish | T → `agrun finish --help` (keep the pp1 sentence) |
| D22 | 143–148 | plan-window bound: how to read "until N % used" vs "consume N" | efc1910 | rp | f, r (l.176–224, full) | — (agbudget) | S → one text for the reading rules (`agbudget --help`); desk keeps "record how you read it in the opening post" |
| D23 | 152–159 | evidence format `[name #id] sender · time`; ✔ thread finished; unreadable/partial thread | 9c002ba | design (front_desk p2) | — (f lacks it; r partly) | — | F+S → shared text: how to read the chatlog/threads |
| D24 | 161–168 | threads are yours; work continues in `workplan-`/`workrun-`; `agentchat read`; reading costs nothing | 9c002ba | as9 (reading free) | f (l.207), r (step 2) | read | T+F+S → head (reading is free) + `read --help` (✔ bare name, `--since`) |
| D25 | 172–188 | Observer's stop post: its three openings, what it contains, addressed to you | 2583175, bcba734 | fs1, fs2 | f, r (identical) | — | S → one shared file |
| D26 | 190–199 | run `agentchat recheck <anchor> --after <ack>` first; verdict list; only the owner's posts count; quote it | b64de4b, 7b42c2c | fs4, fs3A | f, r | recheck | T → `recheck --help` (already has most; **lacks UNOWNED**); F: "act on the verdict" |
| D27 | 201–203 | RESUMED/RESUMING/ASKED → do not post; a second run beside the first is the one wrong move | b64de4b | fs4 (T3) | f, r | recheck | T+S → `recheck --help` (what each verdict means for you) |
| D28 | 204 | FINISHED → say so | b64de4b | fs4 | f, r | recheck | T |
| D29 | 205–208 | UNOWNED → find the real conversation and post there | 7b42c2c | fs5E | f, r | recheck | T (recheck help must gain UNOWNED) |
| D30 | 209–220 | STOPPED → resume in that same conversation; the one case of posting again; resume ≠ agreement | bcba734 | fs2 | f, r | — | F+S → shared Observer text |
| D31 | 221–223 | a task is closed only when its record says `completed` | 19f2bfa | fs3A2 | f, r | — | F+S |
| D32 | 224–227 | work moved only if its owner posted in that conversation after the stop | 7551da6 | fs3A | f (with more detail), r | recheck | F+S (also in `recheck --help`) |
| D33 | 228–233 | a live process or a named wait → do not start it twice; ask its owner | bcba734 | fs2 | f, r | — | F+S |
| D34 | 234–236 | cannot go on / not yours → ask the developer with a response request | 2583175 | fs1 | f, r | send --intent | F+S |
| D35 | 238–241 | Observer counts only fresh work; ten-minute bound | 2583175, bcba734 | fs1, fs2 | f, r | — | F+S |
| D36 | 245–250 | an answer that names you is owed until the receipt; AWAITING_DELIVERY; settled by a decision | 3207a39 | fs6r | f, r | trace, receipt | T → `receipt --help`, `trace --help` |
| D37 | 252–261 | `receipt`, `--repair`, `--because`; a receipt changes no work; never hand-write one (#15362) | 3207a39 | fs6r | f, r | receipt | T → `receipt --help` (subcommand help **lacks** "never hand-write" and "changes no work"; the top-level help has the first) |
| D38 | 265–276 | a person keeps a decision → `hold … --for … --unit … --evidence`; settled holds; `release`; hold vs reserve | 4939026 | fs6h | f | hold, release | T+F → `hold --help` (add hold vs reserve); F: "record a person's hold; nobody acts on held work" |
| D39 | 280–292 | request's standing → `disposition` kinds; never say cancelled/withdrawn succeeded; end your own run with `agrun finish`; reversing | 93a133c | fs6x1 | f | disposition | T+F → `disposition --help` (add "never report as success" and the agrun finish pointer); F: one line |
| D40 | 296–301 | each run ends; no blocking wait; callbacks bring you back | 1be5b8f | as5-8 | f (implicit), r (step 6) | — | F+S → shared (overlaps `agentchat --help` Notes and CONTINUATION_GUIDE: both say it partly) |
| D41 | 303–309 | when delegating say what/where; called back → judge on evidence; ack ≠ done; answer their question with `send` | 1be5b8f, 9c002ba | as5-8 | f, r (step 2) | send | F+S |
| D42 | 311–312 | a post runs the agent; "how is it going?" restarts their job | 1be5b8f | as9 | f (l.207), r (step 5) | — | F+S → head (the cost of speaking) |
| D43 | 314–321 | never open `workrun-`; re-plan in `workplan-`; read before posting; ✔ is finished; `send` refuses resolved | ecf27f9 | fd-wr | f (l.135, argue) | send, read | F+S (the ✔ half is also in `agentchat --help` Notes) |
| D44 | 323–333 | work vs reference; `send` default; `--relation`; `agentchat relation` | 75d739f | fs6x1 | f (identical) | send, relation | T → `send`/`relation --help` (already close); F: "commenting or cleaning up in another request is a reference" |
| D45 | 335–336 | put references (topic, commit, figures, links) in the reply | 1be5b8f | design | — | — | F+S (one line) |
| D46 | 340–342 | begin with the first word of the reply; no preface (seen live twice) | 23012f1, 143908c | fd1, fd2 | r ("entry is the reply") | — | F+S → next to REPLY_GUIDE (all conversational roles) |
| D47 | 346–359 | `agrefs` generic paragraph: list/sync/show/path; work from the named revision; originals read-only; say when a reference disagrees | 244a50c, 70b7f3f | design (adventure_game p2); discovery (give_context_easier) | f, a, archsage, autolab ×3, forge ×4 (see below) | agrefs | T → `agrefs --help`; one line in each guide |
| D48 | 361–363 | resolve a named reference; carry `<source>@<rev>:<path>`; adopt a commit, never `latest` | 70b7f3f | design | f (longer) | agrefs sync | F (role-specific; front's longer wording kept) |
| D49 | 365–367 | a context-panel reference already carries the full commit | 70b7f3f | design | f, a | agrefs | T → `agrefs --help` |

## front (`agent/guides/front/guide.md`, 363 lines): what desk does not have

Rows the two guides share are the D rows above (front repeats D4, D7–D22,
D24–D49 with small wording differences). Only front-only paragraphs are
listed here.

| # | lines | content | commit | evidence | class → destination |
|---|---|---|---|---|---|
| F1 | 2 | reply goes to the developer | 586cc6f | as5-8 | F → head |
| F2 | 4–7 | casual chat → reply; ask for work → read `tools/`; propose "Can I proceed?"; can't → politely say so | 159b92b, 233f6cd | prop (proposal); none (small talk) | = D7/D8/D10 |
| F3 | 37–49 (tail of project) | after autolab answers, hand the first work over as a `workplan-` in the new channel citing the decision; do not re-ask; only on a stated decision | 1419002, 4538f9e | agm1 | F+S → keep, shared (desk lacks it) |
| F4 | 64–70 | execution preference goes into the opening post in the developer's words, with the option name; the run forgets otherwise | 40c48dd | rp | F+S (desk lacks it) |
| F5 | 74–84 | opening a run is the whole reply; why (anchoring mechanism) | 53bbfb9 | rt2 | F+S (= D17, fuller) |
| F6 | 154–160 | execution options: ask, never assume; `options`; unknown ≠ no; never guess a profile | 40c48dd | rp | T → `options --help` (already says all of it) |
| F7 | 162–172 | translate an intent to a published option; `use` is configuration; post the request separately; `default`; translate again for a third agent | 40c48dd | rp | T+F → `use --help` (add "translate again for a third agent"); F: one line |
| F8 | 174–176 | asking another agent to run a way does not change your own run | 40c48dd | rp | F → one line |
| F9 | 178–182 | judge after evidence exists; `trace`, `read`; ask autolab about projects, cagent about the cluster; do not open repositories or nctl | 477c450 (+ older) | robust_workflow p1 | F+S (routing part repeats introductions) |
| F10 | 184–187 | you never run/play/view a delivery; record acceptance as theirs, quoting | 8b4674a | agm2 | F+S (desk lacks it) |
| F11 | 197–206 | record a whole acceptance where the agent's introduction says; acknowledging records nothing; never "I played it" | 99ad106, 8b4674a | agm2, robust_workflow p3 | F+S (overlaps D18) |
| F12 | 207–208 | reading free, posting costs; wait to be named | df8da45 | as9 | = D42 |
| F13 | 209 | already done → reply so | 586cc6f | none | A (= D11) |

front has no D5 (no-mention rule), D23 (evidence format), D40–D43 as a
section, or D46 (no preface). The two guides have diverged in both
directions. This is the drift the braindump suspected.

## routine_run (`agent/guides/routine_run/guide.md`, 284 lines)

| # | lines | content | commit | evidence | class → destination |
|---|---|---|---|---|---|
| R1 | 1–5 | you drive one run; this topic is your record; the reply is an entry | cbafb8a | design | F → head |
| R2 | 9–26 | what you have: chatlog, threads, `tools/agents.md`, the routine guide, `tools/budget.md`, `options` | cbafb8a, efc1910, 40c48dd | design | F → head (one line per item) |
| R3 | 30–40 | read opening post and last entry; judge on evidence (ack ≠ done); decide; the opening post wins over the guide | cbafb8a | design | F (the "opening post wins" rule stays) |
| R4 | 41–53 | honour the execution preference at every delegation; translate per recipient; unknown → say so; do not silently retry | 40c48dd | rp | T+F → `options`/`use --help`; F: every delegation, never another way |
| R5 | 54–60 | send into the entrance; read first; a new topic per delegation; post only with something | cbafb8a, 2583175 | as9, fd-wr | S (= D42/D43) |
| R6 | 61–66 | the entry is the end of the serving; no block for "serving over" | cbafb8a, acea1fc | rt1 | F (role-specific) |
| R7 | 68–75 | accepting when the guide says the runner accepts; reserve → ask | 61ffa8c | fs5 | S (= D18) |
| R8 | 77–81 | never post into `guide`; never open another `routinerun-`; `agrun adopt` | cbafb8a, 5bd9216 | fd1, fs5 | F (+ T for adopt) |
| R9 | 83–154 | Observer section | 2583175…7b42c2c | fs1–fs5E | S (identical to D25–D35) |
| R10 | 156–174 | receipt section | 3207a39 | fs6r | S/T (identical to D36–D37) |
| R11 | 176–224 | conditions about usage: name the window, `+` pools, unknown pool, "exceeds" vs "reaching", account vs run cost, reset, stale read, not a wall, ending ≠ achieving | efc1910, 40c48dd, 9ac28ce | rp | T+S → the reading rules move to `agbudget --help` (desk D22 points there too); the run-specific "every entry says which read" and "ending ≠ achieving" stay |
| R12 | 226–280 | ending the run: serving end ≠ run end; the block is irreversible; "anybody I am waiting for?"; `achieved: false` meaning; the block's shape | cbafb8a, acea1fc | rt1 (three runs lost) | F (role contract; stays whole) |
| R13 | 282–284 | the entry is what you mark as the reply | f7508c0 | design | F (short) |

## argue (`agent/guides/argue/guide.md`, 191 lines)

| # | lines | content | commit | evidence | class → destination |
|---|---|---|---|---|---|
| A1 | 1–6 | who; say what you do in the reply that does it | d07d742, f7508c0 | ag1 | F → head |
| A2 | 10–15 | your part: served on every post; others only when named | d07d742 | design | F |
| A3 | 19–35 | the desire is a human's post; `ag-argue desire:` block | d07d742 | design (listener contract) | F (role contract) |
| A4 | 39–58 | invite by the roster's `bot:` line; a mention costs a run; selectors; never mention the human | d07d742, 03d2682 | ag1 | F (stays) |
| A5 | 62–67 | after an answer: relate, carry on, always reply | d07d742 | design | F |
| A6 | 71–81 | invite archsage once the desire is recorded; reuse; sages directly | dbe0d6b, 03d2682 | design | F |
| A7 | 85–101 | study first vs project; no confirmation round | dbe0d6b | design | F |
| A8 | 105–135 | GOAL.md / RESEARCHPLAN.md contents; `agproject open`/`plan`; archsage for a new study; `status`; nothing is started; never post into `workrun-` | dbe0d6b, 1419002 | design, sp2, fd-wr | T+F → document contents to `agproject open/plan --help`; routing to archsage as D14 |
| A9 | 139–166 | finishing: outcome contents; `ag-argue` complete block | dbe0d6b | design | F (role contract) |
| A10 | 170–183 | agrefs generic | 244a50c, 70b7f3f | design | T (= D47) |
| A11 | 185–191 | argue-specific references; context-panel commit | 244a50c, 70b7f3f | design | F (one line) + T (= D49) |

## The same text in other agents' guides

| paragraph | files | lines each | class → destination |
|---|---|---|---|
| agrefs generic (D47, word for word) | agfront `desk`, `front`, `argue`; archsage `archsage`; agautolab `workrun_supercoder`, `workplan_superdirector`, `argue/role.md`; agforge `assetplan_front`, `assetplan_generator/guide_plan.md`, `assetrun_generator`, `argue/role.md` — **11 files, 4 agents** | 14 | T → `agrefs --help`; each guide keeps its role-specific follow-up paragraph (autolab: `direction/REFERENCES.md`, `agrefs changes`; forge: `--init-image`, `required_items.md`; archsage: references are not findings) |
| agrefs short variants | agobserver `intake`, `argue/role.md` | 4–5 | already Tool Giving; leave |
| context-panel full commit (D49) | agfront `desk`, `front`, `argue` | 3 | T → `agrefs --help` |
| Observer stop section (D25–D35) | agfront `desk`, `front`, `routine_run` | 71 | S → one shared file |
| receipt section (D36–D37) | agfront `desk`, `front`, `routine_run` | 19 | T → `receipt --help`; one line left |
| relation paragraph (D44) | agfront `desk`, `front` | 11 | T → `send`/`relation --help` |
| hold, disposition (D38–D39) | agfront `desk`, `front` | 14, 15 | T → `hold`/`disposition --help` |
| acceptance holder (D18/R7) | agfront `desk`, `front`, `routine_run` | 20 / 8 | F+S |
| study establishment (D14/A8) | agfront `desk`, `front`, `argue` + archsage's own introduction | 14 | routing is the introduction's; one line left |

## What already exists as generated or appended text

- `REPLY_GUIDE` (`agag.reply`, "How your reply is posted", 3 303 chars in
  run-0160) and `CONTINUATION_GUIDE` (`agag.continuation`) are appended by
  `prompt_with_guide` to every conversational role's prompt. D40 partly
  repeats the continuation text. D46 (no preface) is about the reply
  mark's content, so it belongs next to `REPLY_GUIDE`.
- `agentchat --help` Notes already carry D40 ("say what you want and
  finish"), D42 (a post costs a run is implied), D43 (✔ / `send` refuses)
  and the `accept` mechanics of D18.
- `tools/agents.md` carries the introductions, which already hold the
  routing in D14 and F9. `tools/runs.md` carries D19's run list.

## Help gaps found in passing (for step 2)

- `recheck --help` omits **UNOWNED** (the verdict D29 depends on).
- `receipt --help` omits "never hand-write a receipt" and "a receipt
  changes no work".
- `hold --help` does not say how a hold differs from `reserve`.
- `disposition --help` does not say that cancelled/withdrawn are never
  reported as success, or that a routine run of yours is ended with
  `agrun finish`.
- `use --help` does not say that an option name belongs to one agent and
  must be translated again for a third one (`options --help` half-says it).
- The known gaps from the plan still stand: `agproject status` has no
  description; no help lists projects/studies; no help calls the
  channels, sages and topics "the board", readable for free.

## Tally

| class | desk rows | notes |
|---|---|---|
| F (stays, short) | D1, D2, D5, D6, D8, D23, D30–D35, D40–D43, D45, D46, D48 | most become one or two sentences |
| T (to `--help`) | D12, D13, D16, D19–D22, D26–D29, D36–D39, D44, D47, D49 | the guide keeps at most one line each |
| S (write once) | D4, D25–D35, D36–D37, D18, D47 | across desk/front/routine_run and 4 agents |
| A (drop candidates) | **D3, D7, D10, D11** (and front's F13, F2's small-talk half) | D7 is the branch run-0160 took; the other three have no failure behind them |

No Evidence-Driven paragraph falls in class A.
