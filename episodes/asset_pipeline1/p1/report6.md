# Report 6 — agautolab: answer agforge questions on create- topics

Status: **done**. `agautolab` suite green (117 passed, was 103).

This closes the loop Step 4 opened. agforge asks by leaving `question.flag`
and mentioning whoever spoke last; autolab now hears that mention and answers.

## The mention gate is the whole design

`create-` joins `mission-` and `run-` in the sweep filter, and `dispatch`
routes a `create-` topic in a `pj-*` channel to `handle_create`. That handler
reacts **only** when the topic's last message mentions the autolab bot.

The asymmetry is load-bearing. agforge reacts to any `create-` post that is
not its own. If autolab did the same, then agforge's plan registration would
wake autolab, autolab's post would wake agforge, and the two would answer each
other forever — one paid agent run per lap, in both repositories. The mention
is what makes only *questions* wake autolab.

`mentions_us(content, bot_name, self_id)` accepts all three Zulip spellings —
`@**Name**`, the silent `@_**Name**`, and the disambiguated `@**Name|<id>**`.
`apply_markdown: false` means the raw syntax is what arrives. `@**Autolab
Two**` is *not* a match for `Autolab`, because the pattern anchors on the
closing `**`.

Two gates precede any cost:

1. the topic must start with `create-asset_` — agforge's own
   `create-YYYYMMDD-HHMMSS-<id>` topics are none of autolab's business, and
   this check costs not even a history read;
2. the last message must mention the bot, and must not be autolab's own.

Failing either, the handler returns **without posting**. It is then still not
the topic's last poster, so the sweep will look again next time, at the price
of one history read and no agent run. That is the cheap, idempotent
non-reaction the loop needs.

## No ack

`handle_create` is deliberately not `serve_topic`-shaped. `serve_topic` acks
first, and an ack here would make autolab the last poster — which is precisely
the signal that hands the conversation back to agforge. agforge would resume
before the answer existed. The single post at the end is both the answer and
the hand-back.

## The context comes from Plane, and only from Plane

`mission.asset_answer_context(project, work_id)` returns
`(plan.md text, task.md text)`:

- the work id is the topic name's tail after `create-asset_`
- the asset Sub-Work is recomposed into a document — which gives back the
  `task[N].md` the superdirector wrote, minus the `[Asset]` marker that was
  addressed to the registration and never reached Plane
- its **parent's description is `plan.md` verbatim**, so it is recovered with
  `html_to_text` rather than recomposed. This is why Step 2 stores the whole
  file, heading included, as the description: the round-trip is lossless here.

There is no local copy to read. Step 2 deletes `plan.md` and `task[N].md` from
the project folder the moment Plane accepts them, exactly so that one
canonical copy exists and it is this one.

## The run

```
.local/topics/<channel>/<topic>/<N>/answer/
    chatlog.md   the conversation, last message = the question
    plan.md      the parent Work's description
    task.md      the asset Sub-Work
    answer.md    written by the run; the only thing posted back
```

Role `superdirector` (the Step 2 role, sonnet, writable grant), **cwd
`.local/projects/<slug>/`** so `main/`, `direction/` and `devlog/` are all
reachable, guide `agent/guides/create_answer_superdirector/guide.md`. Timeout
is the superdirector's 1200 s. Prompt via the existing `prompt_with_guide`
placement-line pattern, naming the three files and where `answer.md` goes.

A fresh generation `<N>/` per answer, the same rule the other handlers follow:
a previous round's `answer.md` can never be posted twice.

A run that wrote no `answer.md` raises. Nothing is posted, so agforge is not
handed a conversation that has no answer in it.

### The guide

The file existed as an untracked draft (committed in Step 1 for the
`devlogs/` → `devlog/` fix). It was finished here: it now names `chatlog.md`
and says its last message is the question, keeps the `main/` `direction/`
`devlog/` orientation, and — the part that was missing — says to write
`answer.md` **and why**:

> Only "answer.md" is posted back to the asset creator, so anything you want
> them to know has to be in it.

An agent that knows the consequence can decide what belongs in the file. A
bare "write answer.md" would be an unexplained chainsaw.

## The full loop, end to end

```
run- picks an [asset] Work, no agforge Work  → autolab posts the order
                                               (autolab is last poster)
agforge sweeps create-, front + generator    → question.flag
                                             → "@**Autolab** <question>"
                                               (agforge is last poster)
autolab sweeps create-, sees the mention     → superdirector → answer.md
                                             → posts it
                                               (autolab is last poster)
agforge sweeps create-, resumes              → plan.md → Plane Work
Omni Agent fires runcreate- by hand          → delivery + [S3KEY] footer
run- picks the same Work, agforge done       → resign → coding with the URL
```

## Tests (117 passed, +14)

`tests/test_zulip_listener.py`:

- a mentioned question → `whoami → history → context → answer-run → write`,
  the work id parsed off the topic name, the three files written into
  `<N>/answer/`, the run in the project folder, and `answer.md` posted
- a post with **no** mention → `whoami → history` and nothing else
- our own last post is never answered
- a non-`create-asset_` topic is ignored before any lookup at all
- an answer run that wrote nothing raises and posts nothing
- each answer cuts a new generation
- three mention spellings open the gate; four near-misses (including a
  different bot whose name starts the same) keep it shut
- the prompt names all four files and ends with the guide
