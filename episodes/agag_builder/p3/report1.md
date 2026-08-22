# agag_builder p3 — step 1: provisioner account

## Done

- Created `provisioner@agstudio.local` as a normal Zulip user and promoted it
  to Organization administrator.
- Fetched its API key through `POST /api/v1/fetch_api_key` and wrote the
  three-key Zulip environment to the ignored
  `pj-agdev/.local/zulip/provisioner.env` with mode 0600.
- Verified that `GET /api/v1/users/me` succeeds as that identity. Zulip
  reports user id 17, full name `Provisioner`, and role 200 (administrator).
- Confirmed before the change that the user and env file did not already
  exist. The local Zulip 12.2 application container was healthy.

## Planning-time correction

This realm's `Realm.string_id` is the empty string, not `agstudio.local`.
`manage.py create_user --realm agstudio.local` therefore refused without
creating anything. Querying the existing Developer user's realm established
the actual value, and both `create_user` and `change_user_role` succeeded
with `--realm ''`. The realm host remains `agstudio.local:8543`.

No credential value was printed or added to a tracked file.
