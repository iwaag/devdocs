# agag_builder p2 — step 5: live check

All three exercises went through the re-based listeners, 2026-08-22
10:20–10:2x UTC. Nothing that would show a skeleton difference appeared: no
topic served twice, the mention brought Front back, the entrance answered in
autolab's vocabulary.

## autolab — `pj-runsmoke1/workplan-p2-skeleton-file`

Log (`agautolab/.local/out/zulip-listener.log`):

```
10:20:03 serving 'pj-runsmoke1'/'workplan-p2-skeleton-file'
10:20:03 workplan topic 'pj-runsmoke1'/'workplan-p2-skeleton-file'
10:21:34 serving 'work-r-6'/'workrun-task1-r-6'
```

Zulip, 17 s after the request:

> Plan written and the mission started: one task to add `main/skeleton.md` …
> created R-6 … created sub-work R-7 … work channel work-r-6 is ready …
> opened work-r-6/workrun-task1-r-6; post there to start it

Then "Go ahead." in `work-r-6/workrun-task1-r-6`:

```
10:21:34 serving 'work-r-6'/'workrun-task1-r-6'     supercoder writes main/skeleton.md, asks for approval before committing
10:22:13 serving 'work-r-6'/'workrun-task1-r-6'     after "Looks good, commit it."
10:22:32 "Task complete: main/skeleton.md was created, approved by the developer, and committed (commit e04a930) …
          task R-7: commented yes, Done yes; resolving this topic
          recorded r-6-add-skeleton-md-at-the-root-of-main/task-1 in devlog and pushed"
```

`agautolab/.local/projects/runsmoke1/main/skeleton.md` = "Created on
2026-08-22."; `git log -1` there is `e04a930 Add skeleton.md with creation
date`. Topic renamed `✔ workrun-task1-r-6`. Plan → run → Plane → devlog,
end to end, on the skeleton's `run_role` with the v2 grants and
`skip_permissions` decided from the resolved harness.

## autolab entrance — `autolab-agstudio1/p2-list-plans`

```
10:20:24 serving 'autolab-agstudio1'/'p2-list-plans'
10:20:24 entrance topic 'autolab-agstudio1'/'p2-list-plans'
```

Answer at 10:20:43: one line per `workplan-` topic across `pj-foodchain`,
`pj-runsmoke1/2`, `pj-simpleshooter`, each marked finished/not — the new
plan listed as "not finished". Served by `agag.entrance` with
`agent/guides/entrance_front/guide.md` (autolab's), not the built-in guide.

## agfront — `front/front-p2-greet-agecho`

```
10:20:15 (serving, ack)                      Front: "I've posted a message to agecho-agstudio1 … topic 'hello'"
10:20:26 agecho-agstudio1/hello               Front → agecho: "Hi agecho, could you say hello?"
10:20:36 agecho: "@**Front** Hi Front — hello back from agecho-agstudio1! …"
10:20:36 serving mention in 'agecho-agstudio1'/'hello'
10:20:36 mention in 'agecho-agstudio1'/'hello' serves front/front-p2-greet-agecho
10:20:47 marked 'agecho-agstudio1'/'hello' served up to 1351 in front/front-p2-greet-agecho
10:20:47 Front at home: "agecho-agstudio1 already replied — the task is done. … > Hi Front — hello back …"
```

Same exchange as p1 step 4, now with Front's `on_mention` wired through
`listener_main` and its run through the skeleton's `run_role` with the
`agents.toml` grant.
