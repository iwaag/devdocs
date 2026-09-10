# refactor p3 step 1 — cagent's change record moves into its own channel

## What was asked

Replace the Plane registration of `requested_change.md` with a readable record
in a cagent-owned topic, reuse that topic for discussion and results, identify
the request and its return destination with message anchors, make the route
discoverable in cagent's introduction and usable by its listener, and keep
recording a change distinct from executing one.

## What changed

### cagent has an instance, a channel and an introduction

It had none of the three. `cagent_api/instance.py` gives it an `AgentSpec`:
the name is read from the ignored `.local/instance.toml`
(`instance.example.toml` shows the shape, `CAGENT_INSTANCE_NAME` overrides),
and the same name is the channel it owns. The listener's sweep is now
`agag.agent.topic_filter` — **every** unresolved topic in that channel, and
the `cagent-` and `change-` prefixes anywhere else it is subscribed — so the
older route still works and the record topics are served where they live.

`params/intro.md` and `cagent_api/intro.py` post the contract to `#agents`
under `intro-cagent-agstudio1`, with the roster block generated from the
running instance. Until now cagent was the one agent in this realm whose
entrance nobody could read; the roster it publishes says `channel:
cagent-agstudio1`, `prefixes: cagent-, change-`.

### The record is the conversation

`plane.py` is deleted and `change_record.py` holds the same model, read and
written out of Zulip:

| was | is |
|---|---|
| a Plane issue in the `ClusterAdmin` project | a `change-…` topic in cagent's channel |
| keyed `external_id = "<channel>/<topic>"` | keyed by the `[selfnote][change]` note's **own message id** |
| `name` + `description_html` | one visible Markdown post, named by `[selfnote][doc]` |
| — | `[selfnote][origin] <message id>`, the post that asked |
| — | `[selfnote][changerec] <message id>`, written into the origin |
| a Plane state group | `[selfnote][state]`, newest wins |

**Identity is a message id.** The `[change]` note's id *is* the request
(`c5867`), and the origin is remembered as the id of the post that asked for
it. Three things follow, and they are what the plan asked for:

- the record topic is named `change-<stem>-o<origin id>`, so two requests
  raised under one word never merge into one conversation;
- a rename or a resolve does not lose the record — `ZulipClient.message`
  answers with the conversation the anchor is in *now*, and the restatement
  follows it there;
- a deleted anchor is **absent**. A record whose topic was deleted is not
  silently replaced by whatever took its name: the next registration opens a
  fresh record and says `recorded`, not `restated`.

The origin is told where its record is with a **selfnote**, which is never
counted as somebody speaking, so writing the record buys nobody a run — and
replaying a serving restates the change in the same conversation rather than
forking it. Registering from *inside* the record does the same.

### The two branches no longer share a fate

The plan named the defect: `handle_handoffs` registered the change first and
an exception there prevented the observation. `register_change` is now caught
where it is called, reported as its own section, and `required_info.md` still
runs. One post that asks to be told *and* shown gets both answers or one
answer and a stated failure — never neither.

`agent/guides/front/guide.md` says what registering now does, and says that
writing the request down is not carrying it out. That distinction is
unchanged: nothing here reconciles.

### The pyagag pin moved

cagent pinned `bf64bd8`, from before `agag.document` existed. Upgrading to
`218cb31` brings the shared document split and one behavior change cagent
gets for free: `serve_topic` now prefixes each reply with a mention of the
last other speaker, so cagent hands the turn back like every other agent.
Five existing tests asserted the un-mentioned text and were updated.

## Verification

**Live, with no Plane anywhere on the path.** The listener was restarted onto
the new code and reported its sweep as `channel 'cagent-agstudio1' and
prefixes ('cagent-', 'change-')`.

1. **Register.** Message 5864 in `cagent-agstudio1 › p3-record-demo` asked for
   a note that agstudio hosts the local MinIO outbox. The front wrote
   `requested_change.md`; cagent answered
   `recorded c5867 "Add Cluster Note: agstudio is Host of Local MinIO Outbox"
   in #**cagent-agstudio1>change-p3-record-demo-o5864**` and wrote
   `[selfnote][changerec] 5867` into the origin.
2. **Read it back from the channel.** The record topic holds
   `[change] p3-record-demo`, `[origin] 5864`, the statement as an ordinary
   post, `[doc] 5869`, `[state] requested`.
3. **Continue the conversation.** A question posted in the record topic
   (5874) was served there — the `change-` route — and answered in place.
4. **Find the disposition.** `[selfnote][state] accepted` written by the
   Developer's own credential; `read_change(client, 5867, 14)` resolves the
   anchor to `cagent-agstudio1/change-p3-record-demo-o5864`, origin `5864`,
   statement `5869`, state `accepted`, and `client.message(5864)` still
   answers with the origin conversation.

No Plane credential was read and no Plane service was called. `import
cagent_api.topics_serve, change_record, zulip_window, intro` leaves no
`agag.plane` in `sys.modules`.

**Focused tests.** `cagent/tests/test_change_record.py` (11 cases) pins the
record's own decisions: the first registration, what the record knows about
itself, the origin note being a selfnote rather than speech, the anchor being
somebody's speech rather than cagent's ack, repeated registration from the
origin and from inside the record, a reused origin name being a different
request, a resolved record still receiving its restatement, a deleted record
being absent, a change nobody asked for having nothing to anchor to, and a
statement Zulip would truncate being refused. `test_topics_serve.py` pins the
mixed request and the independence of the two branches.

**Suites.** `cagent/tests` 198 passed. mTLS conformance gate 23 passed.

## Limitations

- The record's state vocabulary is `requested` / `accepted` / `done` /
  `rejected` and nothing writes anything but `requested` yet: a disposition
  is a note somebody posts. That is deliberate for this step — the record
  moved, reconciliation did not join it.
- Old Plane `ClusterAdmin` issues are not migrated. This is a breaking change
  in a private experimental environment, as the plan says.
- `nctl agents observe` still requires Plane configuration; that is step 2.
