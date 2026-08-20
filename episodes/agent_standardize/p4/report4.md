# agent_standardize p4 — Step 4 report: Front asks autolab, and a plan comes back

AI-generated (Omni Agent, 2026-08-21).

## Outcome

All four success criteria met, **on the first live run**, with zero agfront
change. Front had never been told autolab exists; it read the `#agents` board,
found the `workplan-` contract in autolab's own words, proposed, waited for
permission, opened `workplan-enemy-spawn-patterns` in `pj-simpleshooter`, and
autolab answered there with a mission plan grounded in the actual game code.

## The conversation

`#front` / `front-p4-autolab-plan` — the Developer's entrance:

| msg | sender | content |
|---:|---|---|
| 709 | Developer | 「simpleshooter っていうプロジェクトに、敵の出現パターンを増やす小さな開発をお願いしたい。まずは計画だけでいいよ。」 |
| 711 | Front | **the proposal** (quoted below) |
| 712 | Developer | 「うん、お願い。」 |
| 716 | Front | "I've sent the plan request to autolab in pj-simpleshooter…" |
| 721 | Developer | 「autolab から返事きてる？読んで内容を教えて。実行はまだしないでね。」 |
| 723 | Front | **the plan, read back and translated** |

Message 711, the proposal:

> Good, pj-simpleshooter exists. Here's my plan:
> I'll open a topic `workplan-enemy-spawn-patterns` in `pj-simpleshooter` and
> ask autolab to draft a plan for adding more enemy spawn pattern variety —
> nothing gets executed at this stage, just the plan. Shall I proceed?

Three things in one sentence, none of which Front could have known before this
phase: the **agent** to address, the **prefix** for this kind of request, the
**channel** it belongs in — and the fourth, that a `workplan-` plans and does
not execute, which it volunteered without being asked.

`#pj-simpleshooter` / `workplan-enemy-spawn-patterns` — the redirect's target:

| msg | sender | content |
|---:|---|---|
| 714 | **Front** | the request, composed by Front in its own words |
| 715 | autolab-agstudio1 | the ack |
| 720 | autolab-agstudio1 | **the mission plan** |

Message 714 is Front's, under the Front bot's identity — not the Developer's,
and not relayed by anything. Message 720 is the plan:

> ミッションの解釈: 現状の敵スポナーは「ランダムXに1〜3体同時湧き＋左右ウォブル降下」の
> 単一パターンのみ。これを、複数の出現パターン（隊列・進入経路・軌道）が時間とともに
> 切り替わる仕組みへ拡張する、という小さめのスコープに絞りました。
> …
> created S2-6 "敵の出現パターンのバリエーションを増やす"
> created sub-work S2-7 … S2-8 … S2-9
> work channel work-s2-6 is ready
> opened work-s2-6/workrun-task1-s2-6 … task2 … task3

That first sentence is the evidence the plan is **grounded**: "random X, 1–3
spawning together, left-right wobble descent" is what `simpleshooter`'s
spawner actually does. autolab read the project, not the request.

It also declined to start, unprompted:

> start.flag はまだ作っていません。依頼者が「実行はまだしないで」と明言しているためです。

## Criterion 4 — nothing fired a `workrun-`

Three `workrun-` topics exist now, and all three are planning artifacts. The
topic listing, with each one's last poster:

```text
work-s2-6/workrun-task3-s2-6  last: 'autolab-agstudio1' (id=11) msg 719
work-s2-6/workrun-task2-s2-6  last: 'autolab-agstudio1' (id=11) msg 718
work-s2-6/workrun-task1-s2-6  last: 'autolab-agstudio1' (id=11) msg 717
```

Every one was written by autolab itself while planning — the task documents —
and the sweep skips a topic whose last post is the bot's own, so all three are
quiet. Nobody else has posted into any of them. The listener's own log agrees:

```text
$ grep -c "serving.*workrun" agautolab/.local/out/zulip-listener.log
0
```

The whole listener log for this phase is four servings, none of them a run:

```text
15:46:08Z agautolab zulip listener starting (pull sweep: all topics in
  'autolab-agstudio1', prefixes ('workplan-','workrun-','assetplan-','bmining-') elsewhere)
15:46:08Z full sweep: 0 awaiting, 14 calls spent, 986 left in the window
15:46:31Z serving 'autolab-agstudio1'/'how-to-request'
15:57:55Z serving 'pj-simpleshooter'/'workplan-enemy-spawn-patterns'
15:57:55Z workplan topic 'pj-simpleshooter'/'workplan-enemy-spawn-patterns'
```

## Criterion 3 — attributability

```text
$ grep -rn "autolab" agfront/src agfront/agent            -> exit 1 (no match)
$ grep -rn "workplan-" agfront/src agfront/agent           -> exit 1 (no match)
$ grep -rn "autolab-agstudio1\|pj-simpleshooter\|agforge-agstudio1" \
      agfront/src agfront/agent                            -> exit 1 (no match)
```

The files those greps cover are the whole of Front: `agent/guides/front/guide.md`,
`agent/zulip_listen.sh`, and four modules under `src/agfront/`.

**agfront's working tree is untouched this phase** — `git status` is empty and
`HEAD` is still `cc36fd0`, p3's commit. Not one byte of Front changed.

## Where the knowledge actually came from

`agfront/.local/topics/front/front-p4-autolab-plan/1/front/tools/agents.md`,
harvested from `#agents` immediately before the first run, generated
`2026-08-20T15:53:40+00:00`. It carries autolab's introduction verbatim, and
the run's whole knowledge of autolab is that text:

```markdown
## autolab-agstudio1

# autolab

This instance develops software projects. …

**Development work goes in the project's own channel, not in mine.** Each
project has a Zulip channel named `pj-<slug>`. Open a topic named
`workplan-<something short about the mission>` there and describe what you
want, in your own words. …

## Choosing the project

Usually the person asking names the project, and the channel is `pj-` plus
that name — a request about a project called `zoo` goes in `pj-zoo`.
```

Front applied that last rule to `simpleshooter` and got `pj-simpleshooter`. The
example in the introduction names a project that does not exist, on purpose,
so the rule is what transferred and not a copyable answer — the leak p2 found
in `agentchat --help`, not repeated.

## The redirect, proven twice over

p4's new thing is that an entrance can **send you elsewhere**. It happened at
both levels:

1. The instance channel redirects. `autolab-agstudio1`/`how-to-request`
   (msg 705 → 706) answers a question with "open a `workplan-…` topic in the
   project's `pj-<slug>` channel", and executes nothing.
2. The introduction redirects *before* anyone knocks. Front never posted in
   `autolab-agstudio1` at all — it read where to go and went straight there.

The second is the cheaper one, and it is the one that scaled: a contract read
off a board cost zero round trips.

## The return path

Asked "autolab から返事きてる？", Front read the topic and reported the plan
back accurately, including the detail that autolab deliberately did not create
`start.flag`. p2 found this capability by accident; here it was used on
purpose, and it is what makes the `front-*` topic a complete conversation
rather than a send-only relay.

The run: `run-0017`, role `front`, profile `sonnet`, harness `claude_code`,
model `anthropic/claude-sonnet-5`, 7 turns, 18.5 s, $0.156.

## The `CLAUDE.md` leak is gone

The memory `claude-md-leaks-into-in-system-agents` warned that
`/Users/eiji/projects/.claude/CLAUDE.md` reaches in-system agents and kills
their runs on a denial. That file no longer exists — `.claude/` holds only
`settings.json`, `settings.local.json` and a lock. Nothing in this phase's runs
quoted it, and no run died on a denial. Worth noting because the hazard was
checked for rather than assumed away.

## Deus Ex Machina note

Every "Developer" message here was written and sent by the Omni Agent with the
Developer's credentials and the Developer's permission in session — same as p2
and p3. A human-written wish would test the same path better; handoff
candidate.

One human decision did enter the loop: the permission classifier blocked
message 712 on its first attempt, work stopped and was reported per
`localrule.md`, and the Developer re-authorized it. That is the harness's
guard on the Omni Agent, not part of the agents' workflow.
