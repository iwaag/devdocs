# Challenge p2 — Step 4: agents, clients and shared guidance

The guides and the Slack bridge now use the common contract. The guide
changes reach the agents at the cutover (step 7), together with the WFE
release that understands them. Before that, a run still served by the old
release would have read commands the old release lacks. The changes are
therefore kept as patches until the cutover (`.local/challenge-p2/guides/`)
and committed there.

## Guides

| Where | Change |
| --- | --- |
| agautolab wfrun guide | The sections *A run that serves a conversation message* and *A run that speaks in a discussion* became one, *A run that speaks in a conversation*. It covers: the request that started your run and the workflow it selected (`converse` alone, `discuss` among several or under the conductor); `wfe conversation show`; `reply` (= `contribute`) with `--re`; a question is `wfe run ask` (WFE posts it, and the reply naming it comes back to your run); a delegate child's question asked from its delegate node and answered on with `wfe delegate answer`; words for work under way; links; speaking from your own mission; starting nobody; long work in your own workflow; integration evidence; openable results; workflow records. Every trial reference is kept. |
| pj-front `converse.md` | Front is the only project and nobody leads. The Developer can ask ponderer to lead in the same conversation, and Front then answers in `discuss` turns. Another project can join with `wfe conversation join` (it starts nothing). |
| pj-front `discuss.md`, `guide.md`, `lesson.md` | One conversation command (`wfe conversation …`); `reply` replaces `contribute`/`discussion` wording. |
| forge, archsage `converse.md` | A child's question is asked from the delegate node. The person's reply naming it comes back to this run, which passes it on with `wfe delegate answer`. Before p2, a new turn carried it. |
| pj-ponderer `conduct.md` | A discussion is a conversation ponderer leads. Facilitation may begin in a conversation with history: its horizon and brief are in `state`, earlier work stays with its runs, and the person may end it (the open run is not stopped; what it awaited is held until resumed). |
| pj-ponderer `guide.md`, `synthesize.md`, `conduct.yaml`, `synthesize.yaml` | The command is `wfe conversation`. A summary goes beside the archive path: `discussions/<id>/` or `conversations/<id>/`. |
| pj-ponderer `converse` workflow + `agent/converse.md` (new) | Direct consultation with ponderer, without leading. It runs on the conversation lane, with no Zulip intake and one talk node, and answers from ponderer's records. Leading stays the person's explicit request. |
| pj-ponderer `tools/discussions` | Reads `discussions/` and `conversations/`, accepts `ag.conversation.v1`, and cites `<path>/transcript.jsonl#n`. |
| pj-aghangout `agent/guide.md` | One conversation contract; the room is `src/room.ts`. |

Each project keeps its own mission and context in a group room. Its
`discuss` guide (or `converse` when it has no `discuss` workflow) is the
one the request selected, and nothing in the selection replaces a
project's own guide.

## Clients

- **Slack** (agdevworld `9c776c0`, release `21a094e` built):
  - it posts `with: front`; the config field `with` replaces `target`, and
    the old `target` is still read;
  - it reads `requests` for its notices ("could not start <participant>'s
    work");
  - user attribution (`via: slack`, the trial author), token
    deduplication, thread ↔ conversation mappings and the aghangout links
    (`?conv=`) are unchanged.

  A room the Developer turned into a group in aghangout stays linked. An
  unaddressed message from Slack then follows the room's routing: to the
  conductor while ponderer leads, to the default addressee otherwise, or a
  refusal the bot explains.
- **WFE Home**: its card lists conversations with their shape (one-to-one,
  several projects, led by ponderer) and links `?conv=`.
- **Request and delegation callbacks**: unchanged. A run's `input.source`
  keeps `{kind: conversation, conversation, message, …, invitation}`.
  `--source-message <conv>:<msg>` works for every conversation.
- **Workflow references and character dialogue**:
  - workflow references follow every conversation's runs;
  - the dialogue worker follows new agent speech in every conversation
    from the cutover;
  - a read queues at most a window;
  - nothing renders history wholesale, and no model runs for migration or
    for a list.

## aghangout

Step 5.
