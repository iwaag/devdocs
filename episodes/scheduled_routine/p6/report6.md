# Step 6 — second localtest run

The second eligible paper was arXiv `2608.23283`, *Apodex 1.1*.  The standard
`autolab project init-localtest 2608.23283 --project studyarxiv` path created
`autodev/studyarxiv-localtest-2608.23283`, cloned it locally, and created its
initial commit (`c4bab45`).  Its completed attempt is commit `c1153db`, pushed
to that local Gitea repository.  The project-level README now links both the
Prime Agent and Apodex localtest folders.

The run independently selected the other `runnable: yes` paper with an
existing manual, then used the documented FrontierAgent quick start.  It
cloned FrontierAgent at `2b82a434…`, installed its Python 3.12 environment,
and launched a one-turn non-interactive ReAct command through the existing
Ollama service.  The source environment built an ephemeral `apodex:local`
Docker image, but the container could not reach the host's Ollama endpoint.
Both `localhost` and `host.docker.internal` attempts ended with the
provider's `Connection error`; the project's `--no-sandbox` fallback rejects
macOS because this workflow requires bubblewrap.

Result: `waiting_external`, with a persistent report and a specific resume
test: make a supported container-reachable endpoint (or documented macOS
native/bubblewrap runtime) available, verify it from a container with a
read-only model-list request, then rerun the documented one-shot.  The
ephemeral image was removed and no container remained.  No routine had to
invent a duplicate repository or restate paper context.

There is no recurring `localtest` schedule.  The two one-shot requests `r10`
and `r11` and their fires `e40`/`e41` are already expired/fired; the standing
routine remains available for a future manual fire.
