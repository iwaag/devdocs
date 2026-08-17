音楽生成をcomfy-uiで行うようにしたい。

pj-agdev/agforge/.local/resources/comfywf/musicに生成成功済みのapi用ワークフローがある。生成成功確認済み。

agforge music generate --prompt でvideoと同様に生成できるようにする。

既存のace step 1.5利用のサービス呼び出しは廃止。agpcのサービスは止まっていると思うが、desired/actual stateから削除、可能なら実体も削除。

pj-agdev/agforge/agent/toolsets/toolset-music.mdはすでに書き換えた。