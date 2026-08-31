# Step 3 — hand the vocabulary over, retire the CLI from the guides

## The guides

`agautolab/agent/guides/workrun_supercoder/guide.md` and
`agforge/agent/guides/assetrun_generator/guide.md` carried the same paragraph,
word for word. Both now carry the plan's replacement:

> When a ComfyUI generation takes minutes, do not wait for it. Submit it, post
> `@**Comfy Notifier** watch <prompt_id>` **in this topic** as a normal
> message, record in your report what is pending and what to do with its
> result, then finish. The notifier reacts to your command, and posts back
> here when the job ends — two lines naming the state and the `prompt_id`;
> read `GET /history/<prompt_id>` yourself for the outputs. Public-channel
> topics only. When *quoting* the command rather than issuing it, put it in a
> code fence.

`grep -rn comfynotify agautolab/agent/ agforge/agent/` now returns nothing:
neither guide mentions the CLI, the binary, its `--help`, or its absolute
path. Guides are read per run, so no listener was restarted for this.

The one sentence kept from the old paragraph is "record in your report what is
pending and what to do with its result" — the notifier says a job ended, it
does not say what the run wanted from it, and that has not changed.

## The docs

- `.local/devenv.md`'s notifier section is rewritten around the command: the
  grammar and its alias, the callback destination being the posting topic, the
  reaction-not-post rule and why, the malformed-command exception, the
  public-channel limit, the code-fence escape, the two launchd env values, and
  the `command-mark.json` high-water mark including what deleting it does.
  The CLI is still documented, demoted to "the daemon's test surface and the
  no-Zulip path".
- The same file's **PATH paragraph is now prefixed with the fact that it is
  moot for consumers** — the guides teach a Zulip post, which needs no binary
  and no environment — and kept, because the CLI path still depends on it.
  Step 2's `launchctl kickstart -k` finding was added to the reload
  instruction in the same section.
- `devdocs/README_DEV.md`'s ComfyUI notifier note gained the two lines the
  plan asked for: the ticket is opened by posting one line in the topic, and
  the ack is a reaction because a bot post in a run topic would serve that run
  early.

## What was deliberately not built

No `#agents` introduction and no entrance for the Comfy Notifier bot. The
braindump says the command syntax is taught per-request in chat for now, and
the plan is explicit: the moment this thing needs an introduction it has
stopped being a tool. It stays a tool.

Done: both guides carry the new paragraph and neither mentions the CLI.
