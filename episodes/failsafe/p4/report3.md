# failsafe p4 — step 3: a post holds 100 000 characters and nothing is cut

## Server

| | before | after |
|---|---|---|
| `max_message_length` advertised by `/register` | 10000 | **100000** |
| source | Zulip's `default_settings.py` | `SETTING_MAX_MESSAGE_LENGTH: 100000` in the deployment's ignored `compose.override.yaml` (the image entrypoint writes every `SETTING_*` into `/etc/zulip/settings.py` at start, as an integer) |
| survives recreation | — | ✓ checked after `up -d` (07:02–07:03Z) and again after `up -d --force-recreate` (07:22Z): `MAX_MESSAGE_LENGTH = 100000`, advertised 100000 |

The generated settings file was not edited. The previous override is kept
beside it as `compose.override.yaml.pre-p4`. Each recreation was about
30 s without Zulip, with no run in flight. The listeners re-registered
their queues by themselves.

## Client (pyagag `e308b94`)

- **Discovery.** `ZulipClient.max_message_length()` asks `/register`
  (`event_types=[]`, `fetch_event_types=["realm"]`, the queue deleted at
  once). The answer is kept in the client and in `<credentials>.limits`
  beside the env file, the way the rate-limit pause is. So a short-lived
  `agentchat` on the same credential asks nothing, and posting makes no
  extra call.
- **Refresh.** After `LIMITS_TTL_SECONDS` (6 h) the next post asks again;
  the setting changes only with a server restart. A mirror restart does not
  need it.
- **Unavailable.** If the server cannot be asked, the last value learnt is
  kept however old. With none at all the client assumes 10000, Zulip's own
  default and the smallest a server ships with, so a post sized to it is
  never cut.
- **Nothing is cut.**
  - `agag.post.compose` no longer cuts (`MAX_CONTENT` and `CUT_NOTE` are
    gone).
  - `send_to_channel` and `send_dm` refuse a post longer than the limit
    **before sending**, with `MessageTooLong(length, limit)`. It is a
    `ZulipRejected`, so delivery treats it as a terminal refusal and never
    retries or cuts.
  - The metadata line is part of the message, so it counts.
- **An agent's reply over one post** is a failed reply, not a cut one:
  - `resolve_reply(limit=…)`; the room is the server limit less the
    listener's own lines under the reply and 300 characters for the
    mention and the `ag-post` line;
  - the run's in-serving repair is told the size it must fit ("keep the
    whole text in a file in your workspace, naming its path");
  - if that still does not fit, the failure line goes out without `end=`
    and the reply stays **owed** (p3's mechanism), so the input is served
    once more with the reason;
  - the unposted reply is kept whole in the serving journal
    (`extra.reply_unposted`);
  - no receipt is written, so nothing reads as delivered.
- **`agentchat send`** over the limit exits 1: "the message is 100001
  characters; this server keeps at most 100000 in one post". A run gets
  the same line as `[selfnote][opfail]` in its home.
- The reply guide no longer states a number of characters. It says a post
  holds tens of thousands of characters, is never cut, and what happens to
  a longer one.

## Audit of other fixed caps

| where | was | now |
|---|---|---|
| agforge `record.POST_LIMIT` (plan documents) | 9000, refused | removed: the client's refusal becomes forge's `RecordError` |
| cagent `change_record.POST_LIMIT` (change statements) | 9000, refused | removed, same way |
| agfront `routine.MAX_REPORT_CHARS` (a routine run's report) | 8000 | the server's limit less 1000 for the delivery's own words (`split_finish(output, limit)`) |
| agentroom chat door | 4000 + "Zulip truncates past 10000" | **kept 4000**: deliberate, a human chat message is not a document. The outdated realm statement and `REALM_MAX_CHARS` are removed |
| agfront `present.MAX_RECORD_CHARS` 8500 | kept: memo records are split into parts by design, nothing is lost. The comment is outdated and noted for step 6 |
| `topics.CONVERSATION_BUDGET` 20000 | kept: a deliberate context-window bound. Its comment now says so |
| `agrefs show` `SHOW_LIMIT` | kept (reading, says where the whole file is) |

## Tests

| suite | passed |
|---|---|
| pyagag | 975 (new `test_post_size.py`, over-long reply owed whole in `test_owed_reply.py`, compose never cuts) |
| agautolab | 324 |
| agfront | 182 (report sized to the server) |
| agforge | 265 (an over-long plan refused by the client) |
| agobserver | 169 |
| archsage | 37 |
| cagent | 204 (an over-long statement refused) |
| agentroom relay | 353 |

The pin `e308b94` is in all seven consumers. All listeners, the gateway,
forge's service, cagent-api and the relay were kickstarted at 07:16:43Z
with no run in flight. Restart recovery started no run.

## Live checks

| check | result |
|---|---|
| Server storage | #13048 (a 52 450-character post: a 2 600-line code block, a sentence after it, `ag-post` line) read back from the server with `apply_markdown=false`: 52 450 characters, no `[message truncated]`, ends with its metadata line |
| Mirror ingestion | Front's, autolab's and Observer's mirrors hold #13048 at 52 450 characters with the sentinel and the line |
| The receiving agent's access | Front, asked for the last row inside the block and the sentinel after it, answered both correctly (#13050: `row 2599 abcdefghij`, `AMBER-7731`). Both are about 52 000 characters in, far past the 20 000-character prompt window. The prompt carries the post's head with its meaning label and says how many characters remain in `chatlog.md`; Front used 6 turns, i.e. it read the complete file |
| An agent's reply > 10 000 with code and trailing metadata | Front's reply #13053: 26 505 characters, a 1 200-line code block, the closing sentence and `ag-post intent=report re=13051 end=13052`, posted whole |
| Over the new limit | `agentchat send` of 100 001 characters: refused before sending, exit 1, nothing posted |
| Limit discovery | `omni-agent.env.limits` written on the first post: `{"max_message_length": 100000, …}` |

The over-long *agent reply* path (repair, then owed, words kept, no
receipt) is proven by the listener-level test on the real journal. It was
not provoked live: a model reply over 100 000 characters is not a small
trial, and the path is the same owed-reply mechanism p3 proved live.

## Cost

Front: two runs, $0.19 (the long question) and the long reply's run. The
cost total is in report6.

## Deus ex machina

None: the Omni Agent posted the test messages as the Developer's stand-in.
