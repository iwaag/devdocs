# Report 4 — agforge: question.flag mention, S3 key in reports, /api/resign

Status: **done**. `agforge` suite green (173 passed, was 159). `/api/resign`
verified live against MinIO on agstudio.

Three unrelated-looking changes with one thread: an asset ordered by autolab
has to survive the gap between "agforge answered" and "autolab reads the
answer". A question that nobody is notified about, and a download URL that
expired an hour ago, are the two ways that gap swallows a request.

## 1. `question.flag` → a Zulip mention

`create_topic.handle_generator` now checks the generator's workspace for
`question.flag` after the run. When it is there, the reply post is prefixed
with `@**<Full Name>**` of the **last non-forge poster in the topic**.

- `last_other_sender(messages, self_id)` walks the fetched message list
  backwards for the first sender that is not this bot and has a display name.
  That list is already fetched for `chatlog.md`, and the realm hides real
  email addresses — a display name is exactly what Zulip's `@**…**` syntax
  wants, so no new lookup is involved.
- Nothing in the answer is parsed. The flag alone decides, which keeps the
  "the front's/generator's *files* drive this, its prose is relayed verbatim"
  rule that the module already lives by.
- With no non-forge poster (impossible in practice — the empty-topic guard
  fires first) there is simply no mention rather than a broken one.

`agent/guides/create_generator/guide_plan.md` now says:

> If you must ask the requester a question instead of planning, write the
> question in your reply and create an empty "question.flag" file — the flag
> is what makes the requester get notified.

The *why* is in the line on purpose. A flag whose effect is unexplained is an
unexplained chainsaw; an agent told what the flag buys can decide whether it
wants that.

This closes agforge's half of the question loop. Step 6 builds autolab's half
— the mention is precisely the gate autolab reacts to.

## 2. The S3 key travels with every delivery

A presigned URL lives `generate.DEFAULT_TTL_MINUTES` (60). The object behind
it lives on. Until now the delivery post and the Plane comment carried only
the URL, so a consumer reading either one 61 minutes later had nothing.

Every delivery now ends with a footer line:

```
[S3KEY] files/2026-08-16/4f5d354d476d47ca9f57c1e28909df2b.zip
```

— in **both** the origin chat post and the Plane comment, because a consumer
may only be looking at one of them, and Plane is the record that is still
there months later. `S3_KEY_MARKER` follows the shape `plane.py`'s `[TOOLS]`
footer already established, which makes it greppable by a consumer that
cannot import agforge (Step 5's autolab is exactly that consumer).

The runcreate summary also names the key it uploaded, so the operator reading
the `runcreate-` topic can re-sign by hand.

### `generate.py` split

`upload_and_presign` was one function that validated the environment, built
the boto3 client, computed a key, uploaded, and signed. Signing an existing
key needed none of that except the first two steps, so it is now:

| function | does |
|---|---|
| `s3_bucket(env)` | validate + client + bucket name |
| `object_key(path)` | `images\|files/<date>/<uuid><suffix>` |
| `presign(env, key, ttl)` | sign an existing key — **no upload** |
| `object_exists(env, key)` | `head_object`, any failure is a "no" |
| `upload_and_presign_key(...)` | upload → `(key, url)` |
| `upload_and_presign(...)` | unchanged contract: just the url |

The last one is kept because the CLI (`agforge image generate`,
`generate.main`) is a different consumer with a different need, and a test
pins that its contract did not move.

## 3. `POST /api/resign`

```
POST /api/resign {"key": "<s3 object key>"}
  200 {"key", "url", "expires_in_minutes"}
  400 body is not JSON, or has no non-empty string "key"
  404 the bucket no longer holds that key
  500 {"error": "misconfigured"} — .local/.env is incomplete
```

Pure script. No agent run, no upload, no cost. This is the correctness device
Step 5 depends on: autolab re-signs immediately before it launches a 1200 s
coding run, so the URL cannot expire mid-run against an unknown remaining TTL.

Two decisions worth naming:

- **The 404 is deliberate.** Presigning is a pure signature operation — it
  succeeds for a key that was never uploaded, and the URL then fails minutes
  later, somewhere the caller has stopped looking. `object_exists` turns that
  silent failure into an answer at the moment it can still be acted on.
- **`SystemExit` is caught.** `generate.load_env` answers a missing
  configuration by exiting, which is right for the CLI it serves and would
  kill this serving thread. It becomes a 500 with the message intact.

`do_POST` now routes on the path rather than early-returning on one route, so
an unknown POST path is still a 404.

## Documentation

Both doors say so: `README_DEV.md`'s HTTP contract block, and the capability
card `service/GUIDE.md` that `GET /guide` serves raw — the card explains the
`[S3KEY]` footer and tells the reader to re-sign right before downloading.
Tool Giving includes the usage information.

## Live verification

Both launchd jobs reloaded (`com.agdev.agforge`,
`com.agdev.agforge-zulip`), then against the running service on `:8092`:

```
GET  /healthz                      -> {"ok": true}
POST /api/resign {"key":"files/nope/none.zip"}
                                   -> 404 {"error":"not_found", ...}
POST /api/resign {"nokey":1}       -> 400 {"error":"bad_request", ...}
```

And a real round-trip: a 22-byte probe zip uploaded through
`upload_and_presign_key` (key
`files/2026-08-16/4f5d354d476d47ca9f57c1e28909df2b.zip`), re-signed through
`POST /api/resign`, and the returned URL downloaded — **HTTP 200, 22 bytes**.
The signature is against `agstudio.local:9100`, as the rest of the deployment
expects.

## Tests (173 passed, +14)

`tests/test_create_topic.py` — mention on `question.flag`; the mention names
the *most recent* asker, not the first; no flag means no mention; and
`last_other_sender` skipping our own lines, an empty history, and a nameless
sender.

`tests/test_runcreate_topic.py` — the `upload_result` stub now returns
`(key, url)`; the origin post ends with the `[S3KEY]` footer, the Plane
comment is `answer\n\n<footer>`, and the summary names the key.

`tests/test_service.py` — `/api/resign` happy path, 404 on an absent key, 400
on three malformed bodies, 500 on a missing configuration, 404 on an unknown
POST route, `upload_and_presign`'s unchanged contract, and `object_key`'s
image/files routing.
