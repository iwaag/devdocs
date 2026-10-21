# argue p5 — Final report: character dialogue as the short reading view

## Where to look

- **The trial discussion in short lines** (the default 短い表示（日本語）):
  <http://agstudio.local:8093/hangout/?discussion=p16-s3-room>
  - Each forge, archsage, argue and aghangout message is a portrait (or
    initials) and a bubble. Selecting a bubble shows the original in
    place.
  - 表示 switches to English or to 原文. 再生表示 plays the same lines.
- **A new message that became a line by itself** (check, closed):
  - <http://agstudio.local:8093/hangout/?discussion=argue-p5-async>
  - <http://agstudio.local:8093/hangout/?discussion=argue-p5-lan>
- **The worker's state**: <http://agstudio.local:8093/wfe/api/dialogue>
- **The cast**: <http://agstudio.local:3000/developer/pj-characters>
  (`manifest.toml`, `characters/*/style.md`). A change there reaches
  aghangout with `npm run characters -- <rev>` and a deploy.

## Revisions

| Component | Revision | Before |
| --- | --- | --- |
| WFE (agdevworld) | `35b3474` | `2eb1976` |
| aghangout (served, with `renderer-current` `dialogue-9`) | `97806f5` | `96d1b99` |
| characters (new managed project, Gitea `developer/pj-characters`) | `aa1e69f` | GitHub `iwaag/agdevworld-settings` `47d5d71`, history kept |
| pj-agdev | `8abde75` | `8e4f45d` |

## Completion evidence

1. **An independent managed character project** (report2):
   - project `characters` holds the reusable definitions, with a
     documented export (`ag.characters.v1`);
   - `tools/characters` adds, updates, normalizes and validates, with
     tests and actionable diagnostics;
   - the `character` workflow adds or changes one character;
   - runtime identity moved to each consumer's mapping;
   - an addition and an update reached an aghangout preview with `src/`
     unchanged;
   - two real style corrections went through it during the trial: archsage
     (`6c7e9f3`) and forge (`aa1e69f`).
2. **A complete discussion reads as concise, coherent character dialogue**
   (report5):
   - p16-s3-room's 14 agent messages read in Japanese: on average 167
     characters against 1,920 in the originals;
   - distinct voices: forge terse and plain, archsage a child with an
     occasional 「じゃ」, the unmapped speakers neutral;
   - every proposal, objection, question to the person and uncertainty
     kept;
   - every bubble was compared with its source.
   - Eight defects were found and fixed:
     - asks dropped;
     - register;
     - nickname;
     - message numbers;
     - suggestions turned into actions;
     - output language;
     - start through a link;
     - recovery after a restart.
3. **Every bubble resolves to its original** (reports 4–6):
   - lines are keyed per message;
   - the toggle keeps place and focus;
   - citations open the original;
   - replies name the original id, on this host and from agpc;
   - drafts and opened originals survive updates.
4. **Display text is outside Git, beside the archive, and outside the
   practical paths** (reports 3, 5, 6):
   - stored in `pj-ponderer/.local/character-dialogue/`, refused unless
     ignored;
   - absent from discussion views, the archive publication, run copies and
     run context (unit checks, and live greps after real turns);
   - `pj-ponderer` is clean.
5. **Missing characters, failures and restarts leave the discussion
   usable** (reports 4, 5, 6):
   - unmapped and portrait-less speakers render plainly;
   - an unavailable renderer showed every original with the reason (live,
     during deployment);
   - failed lines kept their originals and recovered by retry from agpc;
   - a service restart mid-render reclaimed its jobs at once, with no
     duplicate (14 results, 14 keys);
   - original speech is never delayed by rendering.
6. **Works from another LAN PC** (report6): agpc with Chrome passed every
   check, including a new forge message that became its line 6.3 s after
   it arrived.

## Measured

Model `sonnet` through Claude Code, with costs as Claude Code reports them
(`costBasis: list`):

| Measure | Value |
| --- | --- |
| Time per line | about 6–8 s (mean 6.4 s Japanese, 7.5 s English) |
| Cost per line | about $0.014; ~97 % sonnet, the rest a small haiku call Claude Code adds |
| Whole trial, both languages | 28 lines in 98 s with 2 at once |
| New agent speech | its line was ready 8.3–8.8 s after the message on this host, 6.3 s after it appeared on agpc |

`haiku` was about ten times cheaper at the same latency, but it broke the
rules and misread names; it was rejected on that evidence (report3).

## Remaining limitations

- **Adherence is probabilistic.** The final reading still has one message
  number (#152) and one run name (#138), and #132 is 269 characters.
  English lines run longer than aimed (mean 381 characters).
- **Each line is one CLI process** (about 5 s of start-up). Fine at this
  volume; batching would need a context rule that keeps later speech out.
- **Neutral voices.** Only front, forge, archsage and ponderer are mapped;
  ponderer has no portrait or lore, so its plate shows initials. Other
  projects speak in a neutral voice until someone adds a character and a
  mapping.
- **Styles are a starting point.** The `style.md` files were derived by
  Omni from the Developer's lore and are marked as such. The Developer may
  want to rewrite them.
- **Not everywhere yet.** Lines appear in aghangout's meeting rooms only:
  not in one-partner rooms, Slack or WFE's run views.
- **No retention policy.** Older renderings are kept in the store. Deleting
  the store is always safe, because lines are rendered again on demand.

## Step reports

- [report1](report1.md): inventory and contracts;
- [report2](report2.md): the character project;
- [report3](report3.md): generation, cache and isolated reads;
- [report4](report4.md): the reading experience;
- [report5](report5.md): one complete discussion, read;
- [report6](report6.md): deploy, verify and report.
