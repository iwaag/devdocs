# agent_standardize p1 — Step 1 report: rename the accounts, seed the identity file

AI-generated (Omni Agent, 2026-08-20).

## What changed

`agforge`'s instance now has a name — `agforge-agstudio1` — and it is spelled
the same in all three places that can carry it.

### Zulip

`PATCH /users/13` with `full_name=agforge-agstudio1`, using the realm admin
credentials (`pj-agdev/.local/zulip/developer.env`). Success on the first
form; the `/bots/13` fallback was not needed. The bot's email is unchanged,
so every existing message, subscription and DM thread still keys on user 13.

Before / after, from `GET /users`:

```
13 'Forge'  ->  13 'agforge-agstudio1'
```

### Plane

The `agforge@agstudio.local` member (`4f77a23d-…`) keeps its account, its
token and its authorship of every existing issue; only its shown name moved:

```
display_name 'agforge' -> 'agforge-agstudio1'
first_name   'agforge' -> 'agforge-agstudio1'
```

`first_name` came along because Plane's UI shows it in places `display_name`
does not reach; `email` and `username` are untouched on purpose — they are
what the issues and comments hang off.

Done through the API container's Django shell, the same ritual as
`agent_intent/p1/report2.md`, because Plane CE here has no email delivery:

```sh
docker exec -i plane-app-api-1 python manage.py shell < rename.py
```

`PLANE_AGENT_SLUG` in `pj-agdev/.local/plane/agforge.env` is now
`agforge-agstudio1`. The file stays mode 0600 and git-ignored; no other key in
it changed and no value was printed.

### The identity seed

One file the code reads, one key in it:

- `agforge/.local/instance.toml` — `name = "agforge-agstudio1"` (git-ignored)
- `agforge/instance.example.toml` — the committed example
- `agforge/src/agforge/instance.py` — `instance_name()`
- `agforge/tests/test_instance.py` — three cases, passing

The real name is local-only rather than committed because the label is the
hostname, and `devdocs/README_DEV.md` forbids local machine information in
tracked files. With no local file `instance_name()` returns the plain
`agforge`: wrong for an instance, but wrong out loud instead of empty.
`AGFORGE_INSTANCE_NAME` overrides the file, so a second instance on one host
needs no second checkout.

Kept to the name, as the plan asked — this is v1 of the self-definition file,
not the schema.

## Name-shape check

`agforge-agstudio1` was checked against the separators it has to live beside:

- Zulip full names accept `-`; the rename returned `success`.
- Plane display names accept `-`.
- A Zulip channel may be named `agforge-agstudio1` (Step 2 proves it).
- Topic prefixes here are `create-`, `runcreate-`, `intro-` and the resolved
  `✔ ` marker. `intro-agforge-agstudio1` parses as prefix + name with the
  first `-` after `intro`; nothing splits on a later `-`, so the hyphen inside
  the instance name is not ambiguous with the prefix separator.

## Not done in this step

- No channel exists yet — Step 2.
- The listener still knows nothing about the instance name; nothing imports
  `instance.py` yet — Steps 3 and 4.
- The Zulip `Forge` bot's *email* (`forge-bot@…`) still reads `forge`. The
  realm hides emails and keys on numeric ids, so this is cosmetic and was
  deliberately left alone rather than risking history.

Renamed accounts and seeded an identity file that agforge could arguably
maintain for itself — handoff candidate.
