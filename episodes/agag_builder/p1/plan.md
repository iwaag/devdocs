# agag_builder p1 plan — `agag init` が生む薄皮

Goal: 新しい agag 準拠エージェントを **1 コマンド + 数問** で立ち上げ、`#agents` に
自己紹介を投げ、agfront がその紹介だけを頼りに話しかけて返事が返るところまで通す。

Success criteria:

1. `agag init <agent>` が対話で instance 名・topic 接頭辞・role を聞き、
   `pj-agdev/<agent>/` に動くプロジェクトを生成する。
2. 生成物は **薄い**: 固有コードは instance 名・role 定義・guide・`params/intro.md`
   程度。listener / intro / selfnote / entrance / role_run の実体は pyagag 側にある。
3. 生成したダミーエージェント（`agecho` 等）を `listen.sh` で起動し、`#agents` に intro
   を投稿、agfront に「`agecho-agstudio1` に挨拶して」と頼んで返事が `front-*` に届く。
4. agforge を新しい pyagag 骨格に載せ替えて今まで通り動く（他の既存エージェントは
   本フェーズでは触らなくてよい。後方互換は不要、agforge が壊れたら直す）。

Decisions already made (braindump + discussion):

- **テンプレートよりライブラリ。** `agag init` の価値は生成物の小ささで測る。
  規約変更（p8 の selfnote のような）が pyagag の 1 push で全エージェントに届く形にする。
- 新コマンドは `agag`（pyagag の `[project.scripts]` に追加）。`agentchat` はそのまま残す。
- 初版のデプロイはローカル `listen.sh` 起動まで。launchd plist / ansible は生成しない
  （`pj-agdev/devenv/launchd/*.plist.in` を見れば手で 5 分）。
- Zulip の bot アカウント作成、Plane アカウント、チャンネル説明文の編集は人間の作業。
  `agag init` はそれを **最後にチェックリストとして印字** するだけ。agent には
  HTTP 400 で編集できない（standardize p10 TODO）ので無理に自動化しない。
- Plane 登録は「plan 系 topic が Work を登録する」流儀を踏襲するが、**初版のダミー
  エージェントは Plane を使わなくてよい**。Plane 連携は生成物の opt-in
  （`agents.toml` か role の `requires`）にする。

Constraints: secrets は `.local/`。pyagag を変えたら push → 消費側で
`uv lock --upgrade-package pyagag` → push（localrule.md）。料金は気にしない。

## 現状の事実（計画時に確認済み）

- pyagag（`/Users/eiji/projects/pyagag`）: CLI は `agentchat` のみ
  （`agag.chat:main`、`send/read/topics/channels/resolve`）。scaffold は無い。
  共有できている部品: `agag.zulip`（`ZulipClient`, `sweep_serve`, `serve`）、
  `agag.topics.serve_topic`（ack→handler→必ず返事、p10 の共通 seam）、`agag.intro`
  （`post_intro`, `write_agents_md`）、`agag.selfnote`, `agag.harness.run_harness`,
  `agag.agent_config`（`ag.agent-config.v1`）、`agag.plane`, `agag.instance.instance_name`, `agag.status`。
- 各エージェントに **コピペで残っているもの**（pyagag に上げる候補）:
  - `instance.py`（agforge 40 行 / autolab 45 行、差分は docstring だけ）
  - `intro.py`（27 / 34 行、同上）
  - `role_run.py`（forge 205 / autolab 196 / front 133 行）: `agents.toml` 解決 →
    `AGENTCHAT_ZULIP_ENV` / `AGENTCHAT_HOME` / PATH に `agentchat` を通す → `run_harness`。
    `ROLE_ALLOWED_TOOLS` テーブル（role ごとの `--allowedTools`）もここ。
    **grant が無い role は claude_code が permission prompt で timeout まで止まる**
    （agfront/role_run.py の注記）。骨格化するときこの表を agents.toml 側に移すと楽。
  - `zulip_listener.py`（forge 119 行）: `topic_filter`（自分のチャンネル全部 + 接頭辞）、
    `dispatch`（接頭辞→handler、それ以外→`entrance_topic`）、`main`（`sweep_serve`）。
    autolab の 1254 行版は mission 固有が混ざっているので参考にしない。
  - `entrance_topic.py`（130 行）+ `agent/guides/entrance_front/guide.md`（8 行）:
    どのエージェントも同じ「自分の板を読んで答える front run」。guide 内の
    `assetplan-/assetrun-` だけが固有。
  - `anchor.py`（`[selfnote][work]`）、`plane.py` のラベル/プロジェクト決め。
- パス規約: `<root>/.local/zulip.env`（bot 資格情報）、
  `pj-agdev/.local/plane-credentials.env`（共有。forge は `ROOT.parent/.local/…` で
  探しているため pj-agdev 外に置くと見つからない。骨格では `AGAG_PLANE_ENV` 環境変数か
  `<root>/.local/plane-credentials.env` で解決し、親ディレクトリ相対をやめる）、`<root>/.local/instance.toml`、
  `<root>/.local/agents.local.toml`。
- pyagag の消費は git 依存: `pyagag = { git = "https://github.com/iwaag/pyagag.git", branch = "main" }`。
- `agautolab/init_project.py` / `project_init.py` は **autolab が作業プロジェクトを
  作るもの**で、エージェントの scaffold ではない。ただし対話 + Zulip チャンネル作成 +
  Plane プロジェクト作成の実装例として読む価値あり（`client.create_channel`,
  `agag.plane.create_project`）。
- `devdocs/episodes/agent_standardize/p10/report4.md §8` に未解決 TODO 一覧。
  本フェーズで拾わなくてよいが、「自チャンネルの説明文を agent が編集できない」は
  init のチェックリストに直結する。

## Step 1 — 共通化の表を作る（成果物: `p1/skeleton_map.md`）

agforge / agautolab / agfront の `src/` を横に並べ、各モジュール・関数を
**pyagag に上げる / 生成テンプレートに残す / そのエージェント固有** の 3 列に振り分ける。
上記「現状の事実」が叩き台。forge を基準にし、autolab は差分確認だけ。

ヒント: `diff <(sed s/agforge/X/g …) <(sed s/agautolab/X/g …)` で instance.py と
intro.py は docstring 差しか無いことが確認済み。role_run は 3 つとも読むこと。

## Step 2 — pyagag に骨格を入れる（`agag.skeleton` でも `agag.agent` でも命名は自由）

Step 1 の「上げる」列を移す。目安となる API:

- `AgentSpec`（または toml）: `agent`（短名）, `plan_prefix`, `run_prefix`, roles,
  `root: Path`。`instance_name()` は `agag.instance.instance_name(root/.local/instance.toml, fallback=…)` を呼ぶだけ。
- `listener_main(spec, dispatch)`: forge の `zulip_listener.main` + `topic_filter` を一般化。
  `dispatch` が None の接頭辞は `entrance` に落とす。
- `run_role(spec, role, prompt, workspace, …)`: role_run の共通部。
  `ROLE_ALLOWED_TOOLS` は `agents.toml` の `[roles.X] allowed_tools = "…"` に移して
  `agag.agent_config` で読む（`docs/agent-config-v1.md` を更新、schema は v2 にしてよい）。
- `entrance.handle(spec, client, channel, topic)`: forge の `entrance_topic` を移し、
  guide 本文は pyagag 内蔵のデフォルト + `{plan_prefix}/{run_prefix}` 置換。
  エージェント側に `agent/guides/entrance_front/guide.md` があればそちら優先。
- `intro.main(spec)`: `python -m <agent>.intro` の中身。

自由裁量: モジュール分割、名前、dataclass か toml か。禁止事項は **無し**。
既存 tests（pyagag、agforge 19 本）は壊れたら直すか消す。

確認: forge の `zulip_listener.py` / `role_run.py` / `entrance_topic.py` / `intro.py` /
`instance.py` を骨格呼び出しに置き換え、`uv run pytest` と `AGFORGE_ZULIP_LOG_ONLY=1`
での起動、実 Zulip で `assetplan-` 1 本を通す。commit → push（pyagag、agforge、lock 更新）。

## Step 3 — `agag init`

`pyagag/src/agag/init.py`、`[project.scripts] agag = "agag.cli:main"`（`agag init` のみで可）。

質問は最小限（デフォルト付き、`--yes` で全部デフォルト）:

1. agent 短名（引数）
2. instance 名（default `<agent>-<hostname>1`、`.local/instance.toml` に書く）
3. plan / run 接頭辞（default `<agent>plan-` / `<agent>run-`）
4. roles（default `front`）、profile（default `sonnet`）
5. 出力先（default カレントディレクトリ直下 `./<agent>`、`git init` だけして remote は付けない）

生成物（目安、減らす方向で）:

```
<agent>/
  pyproject.toml            # pyagag git 依存、scripts
  agents.toml               # ag.agent-config + allowed_tools
  instance.example.toml
  params/intro.md           # {instance} 置換、接頭辞入り雛形
  agent/guides/<plan>_front/guide.md   # 8 行程度の雛形。「何をするか」は TODO で空欄
  src/<agent>/__init__.py
  src/<agent>/listener.py   # 10〜20 行: spec を定義して agag の listener_main を呼ぶ
  service/listen.sh         # forge のものをほぼコピー
  .gitignore                # .local/
  .local/instance.toml      # 回答から生成
```

終了時に印字するチェックリスト（人間作業）:

- Zulip: bot アカウント作成 → `.local/zulip.env`（`ZULIP_URL/EMAIL/API_KEY`）。
  `<instance>` チャンネル作成は `ZulipClient.create_channel` で自動化してよいが、
  `#agents` 購読と説明文は人間。
- Plane: 使うなら `pj-agdev/.local/plane-credentials.env` が既にある。アカウント追加は人間。
- `uv sync && uv run python -m <agent>.intro` → `service/listen.sh`。
- 常駐させるなら `pj-agdev/devenv/launchd/` の plist.in を複製。

ヒント: テンプレートエンジンは不要。`string.Template` か f-string で十分。
ファイル内容は pyagag パッケージ内に `templates/` として同梱（`importlib.resources`）。

## Step 4 — ダミーエージェントで通す

置き場所はワークスペースルート `/Users/eiji/projects/agecho/`（remote 無し）。
`pj-agdev/` 配下は各エージェントが submodule なので、使い捨てはそこに入れない。

`agag init agecho` → 上記チェックリストを実行 → `listen.sh` 起動 →
`python -m agecho.intro` → agfront に「agecho-agstudio1 に挨拶して、返事を教えて」。

期待: agfront は `tools/agents.md`（intro の harvest）だけで `agecho-agstudio1`
チャンネルの plain topic に投稿し、agecho の entrance（front run）が返事、
agfront が `front-*` に報告。何も固有実装をしていないのに会話が成立すれば成功。

失敗したら直すのが本題（Failure Farming）。詰まりやすい所:
- agfront 側の `tools/agents.md` は run 開始時に harvest される。intro 投稿後に
  Front を呼ぶこと。
- `#agents` に bot が購読していないと intro が投稿できない。
- entrance の guide がデフォルト内蔵のままでも返事は返るはず。返らないなら
  `allowed_tools` の grant 漏れ（permission prompt で timeout）を疑う。
- `[selfnote][rootchat]` は agfront の `agentchat send` が書く。agecho 側は
  `serve_topic` の `reply_to` 経由で自動処理されるはず。

## Step 5 — 記録

- `p1/report.md`: 生成物の行数、Step 4 の Zulip ログ抜粋、失敗と修正、
  pyagag に上げきれず agforge に残ったものの一覧（次フェーズの種）。
- `devdocs/README_DEV.md` の In-System Agents に「新エージェントは `agag init`」を 1 行。
- agecho は残してよい（以後の standardize 実験の最小フィクスチャになる）。
  消すなら Zulip チャンネルは archive、Plane は放置でよい。

## Out of scope（次以降）

- autolab から「新しい agag エージェントを作って」で `agag init` を呼ばせる
  （Step 3 の `--yes` とチェックリスト印字はそのための布石）。
- agautolab / agfront の骨格載せ替え。
- launchd / ansible の生成。
