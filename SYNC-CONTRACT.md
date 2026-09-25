# 名札（Koseki）と台帳（Ledger）の同期契約

> **都度同期**。名札（各道具の `id.koseki.json`）と台帳（`.journal/koseki.ledger.jsonl`）は、情報交換するたびに最新状態を相互に反映する。

## 構成

| 層 | ファイル | 役割 |
|---|---|---|
| 名札 | `{id}.koseki.json` | 各道具が自己保持するメタデータ |
| 台帳 | `.koseki/koseki.ledger.jsonl` | 弁慶が管理する全体名簿（1行 = 1 名札） |
| スキーマ | `koseki.schema.json` | 名札・台帳両方の構造定義 |

## 同期プロトコル

### 1. 双方向ステートマシン

```
名札変更 ──→ dirty=true ──→ 台帳へ push
　↑　　　　　　　　　　　　　　　　│
　└── 台帳へ pull ←── dirty=true ←┘
```

- **名札 → 台帳**: ローカル `koseki.json` を編集したら `dirty=true` → 台帳へ反映
- **台帳 → 名札**: 弁慶が台帳を更新したら `dirty=true` → 名札へ反映
- `last_sync_at` と `sync_hash` が一致したら `dirty=false`（同期完了）

### 2. コンフリクト解決

| 状況 | 処理 |
|---|---|
| 名札と台帳両方 `dirty=true` | **version が大きい方が勝つ**。同じ場合は台帳優先（弁慶の判断） |
| 名札に `ledger` セクションなし | 初回同期とみなし、台帳へ追加 |
| 名札が物理削除された | 台帳の `status` を `archived` に更新（除籍） |

### 3. 同期トリガー

- `soubi` TUI 起動時（全体スキャン）
- `soubi sync` コマンド実行時
- 名札ファイル保存時（inotify / ファイル監視）
- 弁慶エージェントの定期巡回（`/skill:benkei sync`）

### 4. ハッシュ計算

```
sync_hash = sha256(id + name + type + status + repo_url)
```

内容が変わればハッシュが変わり、dirty 判定が行われる。

## 権限

- **読み取り**: 誰でも（名札・台帳両方）
- **名札書き込み**: 各道具のオーナー・開発者
- **台帳書き込み**: 弁慶（統括）のみ
- **コンフリクト仲裁**: 弁慶

## 拡張予定

- `ledger.trust_score`: 道具の信頼度スコア
- `ledger.usage_count`: 使用回数（opencode / pi の呼び出しログから集計）
- `ledger.health`: 最終動作確認日時
