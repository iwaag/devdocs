# Step 3 — first local reproduction attempt

The first production `localtest` run selected arXiv `2608.23552`, *Prime
Agent: A Self-Improving RLM Harness*. `2608.23283` was not selected because
its documented self-hosting path requires an NVIDIA GPU; the selected paper
has a smaller credible install-and-one-shot path with no local model weights.

## Repository and task timeline

| UTC | event |
|---|---|
| 06:16:41 | production dispatcher fired `e40` |
| 06:17 | Front opened `#pj-studyarxiv` › `workplan-localtest` |
| 06:21 | Autolab created mission `S3-1` and tasks `S3-2`–`S3-4` |
| 06:21–06:24 | task 1 created and documented the repository |
| 06:24–06:28 | task 2 installed, exercised, and removed Prime Agent |
| 06:28–06:30 | task 3 recorded the evidence and `waiting_external` state |

`localtest-2608.23552/` was created through `autolab project init-localtest`
and is backed by `autodev/studyarxiv-localtest-2608.23552`. It is committed
and pushed at `fbab1ce` after a portable-evidence cleanup. The study project's
README now points to the local-test folder; the test holds its own README,
state file, and report, so a later run does not need chat history.

## Experiment

Target: Prime Agent release `v0.8.1`, whose release tag resolves to commit
`514633727bf26d74f39f3119c2b0e31a5ceb2a9d`.

The official installer completed, verified the release checksum, and provided
`prime-agent --version` = `0.8.1`. A one-shot invocation was then attempted
in a disposable scratch directory without inventing a provider credential. It
exited 1 with `No API key found for the selected model.`; stdout was empty.
This is useful boundary evidence: install and CLI startup work, while a real
model response cannot be tested without a provider credential.

The npm-global package and disposable scratch directory were removed after
the evidence was captured. No container or persistent service was created.

`localtest.yaml` is `waiting_external`. The repository's persisted handoff
asks an upper actor to supply or select a supported provider API key; the
read-only completion proof is a one-shot response on stdout with exit 0.
No credential was copied, created, or repurposed.

## Live findings

- The initial task commits were not pushed by the direct task close-out path;
  the final Autolab task pushed the accumulated repository history. This was
  verified by the final branch being level with `origin/main`.
- The first generated experiment report included machine-specific paths. They
  were removed before the final push, leaving portable commands and evidence.
- Did this cleanup work for Autolab — handoff candidate.
