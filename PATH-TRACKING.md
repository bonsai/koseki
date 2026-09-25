# 引っ越し追跡（Path Tracking）

> `.soubi` は動く。道具は引っ越す。戸籍は連続する。

## 問題

- GH repo `bonsai/koseki` とローカル `.soubi/.koseki/` が二重化している
- ローカルで `mv` すると、過去のパスが失われて「あの道具、どこに行った？」になる
- 名札（`id.koseki.json`）は個別ファイルなので、引っ越しても持ち歩けるが、台帳（`ledger`）は集約 DB なので引っ越し痕跡を記録する必要がある

## 設計

### 1. 名札（各道具の `id.koseki.json`）

```json
{
  "id": "skill.observe.kankyou-hub",
  "path": ".skills/observe/kankyou-hub",
  "moved_from": [
    ".skills/old-observe/kankyou-hub"
  ],
  "root_base": "~/.soubi"
}
```

- `path` — **現在**の `.soubi` からの相対パス
- `moved_from` — 過去に存在したパスの配列（時系列順）
- `root_base` — `.soubi` 自身が引っ越した場合のルート記録

### 2. 台帳（`.koseki/koseki.ledger.jsonl`）

名札からの同期時に `path` と `moved_from` を吸い上げ、ledger 各行に付与。

### 3. 引っ越し手順

```bash
# 1. 旧パスを記録
jq '.moved_from += [.path]' old/id.koseki.json > tmp.json

# 2. path を更新
jq '.path = "new/path"' tmp.json > new/id.koseki.json

# 3. 旧名札を削除し、新名札を配置
rm old/id.koseki.json
mv new/id.koseki.json new/path/

# 4. 弁慶に同期を依頼
/skill:benkei sync
```

### 4. GH repo との対応

| 場所 | 役割 | 同期方向 |
|---|---|---|
| **ローカル `.soubi/.koseki/`** | 正本（Master） | ← 編集はここ |
| **GH `bonsai/koseki`** | バックアップ・公開参照 | → push で反映 |

`.soubi/.koseki/` は `.soubi` の一部として常にローカルに存在する。GH は clone/subtree ではなく、単なる `origin` リモート。

引っ越し時は `root_base` を更新し、ledger に `root_base_history` を追加予定。
