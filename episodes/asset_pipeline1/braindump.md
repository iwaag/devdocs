アセット発注から組み込みまでへのパイプラインを整理する

# 現状

## レイヤー

### create-トピックでの発注
現状Omni Agentが発注している
autolabがどう自己判断でアセット仕様を決めて発注するかというフローはほぼ無し

### agforge
create-トピック発火
front... チャットログから要望を抽出
generator... 計画を作成、Plan work化

runcreate-トピック発火
generator... 計画を実行、result作成
スクリプト... zip化して一時url作成

### agautolabの受け取り、組み込み
無し
