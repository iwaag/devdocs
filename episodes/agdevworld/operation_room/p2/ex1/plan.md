# operation_room p2追補 — done行のconfirm(手動クリア)

p2のボードでは、relayが✔リネームを目撃した行が `done` として残り続ける
(スイープは✔トピックを読まないので、doneは目撃した遷移のみ。relay再起動まで
無期限に溜まる)。これを**人間が「見届けた」と宣言して消せる**ようにする。

方針: 時限evictionではなく手動confirm。このボードの原則
「証拠は人間が見るまで残す」と一致させる。**まとめて全部消せることが主要件**。

## relay側: `POST /ops/confirm`

- ボディ無し(または `{"all": true}`)= 現在の全done行をconfirm。
  個別指定(`{instance, channel, topic}`)は任意実装 — 全消しだけでも要件は満たす。
- confirm記録はrelayのインメモリに置く。done行の本体と寿命が揃い
  (再起動で両方消える)、ブラウザ間でも一貫する。永続化しない。
- これはrelay自身のメモリへの書き込みであり、Zulipへは何も書かない。
  p2制約5(観測者はZulipに投稿しない)は無傷。GET-only窓にPOSTが1本
  増えるが、非公開loopback運用なので認証は不要。
- **実装ヒント(裁量)**: confirmを `(channel, bare_topic)` だけでなく
  その時点の最新message idと組で記録すると、「confirmはその時点の姿を
  隠すだけで、以後そのトピックに新しい投稿(unresolve含む)があれば行が
  自然に再浮上する」という性質が無料で手に入る。単純削除
  (`_topics` からのdel)は、後のunresolveリネームが `_apply_update` で
  old_key不明→無視になり再開を見落とすので避けたほうがよい。

## 消せる行の制限(これだけは守る)

confirmできるのは **`done` の行だけ**(`stale_state == "done"` の
unknown行を含めてもよい)。`stalled` / `awaiting` / `acked` は消せない —
生きている負債を視界から消せるボタンは、p9の「26分誰も気づかない」を
ボタン1つで再現する装置になる。relay側でも状態チェックして拒否する
(フロントの出し分けだけに頼らない)。

## フロント側

- チップ行に「✓ confirm N done」ボタンをdone行が1件以上あるときだけ表示。
  押すと `POST /ops/confirm` → reload。
- 行単位のボタンを付けるかは裁量(`row.actions` の仕組みがtasksビューに
  先例あり)。付ける場合、agent_room step 5が残した
  「actionボタン付きカードはスクリーンショット未検証」の穴をこの検証で塞ぐこと。
- **同時に直す**: サブタイトルの「N rows open」のカウントからdoneを除外する
  (openという語と実態のずれの解消。p2レビューで指摘済みの小修正)。

## 検証

`#ops-testbed` でp2 step 5のレシピを流用:
人工stall → 返信/✔で `done` → confirm allで消える →
(message id方式を採ったなら)同トピックへ追加投稿して行が再浮上する、まで
一巡をスクリーンショットで確認。ボタン付きカードの見た目も撮る。

## 制約(最小限)

1. confirmできるのはdone行のみ。relay側でも拒否する。
2. confirm記録はインメモリのみ(ファイル永続化しない)。
3. Zulipへの書き込みは発生させない。

他(エンドポイントの形、個別confirmの有無、ボタンの見た目・文言)は裁量。
