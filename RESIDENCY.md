# 居住ルール（Residency）

> 道具は、生まれたタイプによって、住める場所が決まっている。
> 無秩序な散乱は、孤児の原因となる。

## 居住可能エリア

| type | 住める場所 | パターン | 例 |
|---|---|---|---|
| **skill** | `.skills/{category}/{name}/` | `.skills/*/*/` | `.skills/observe/kenja/` |
| **agent** | `.agents/{role}/{name}.md` | `.agents/*/*.md` | `.agents/統括/bonsai.md` |
| **deploy** | `.deploy/{name}/` | `.deploy/*/` | `.deploy/yose-db/` |
| **ontology** | `.ontology/{code}/` | `.ontology/*/` | `.ontology/xX/` |
| **crx** | `.crx/{name}/` | `.crx/*/` | `.crx/chatgpt-crx/` |
| **extension** | `.extension/{name}/` | `.extension/*/` | `.extension/soubi-tui/` |
| **command** | `.command/{name}` | `.command/*'` | `.command/deploy-prd` |
| **plugin** | `.plugin/{name}/` | `.plugin/*/` | `.plugin/vocab-checker/` |
| **other** | `.journal/` `.command/` など | 個別判断 | `.journal/2026-09/` |

## 禁止区域

- **skill は `.agents/` に住めない** — type が違う
- **agent は `.skills/` に住めない** — 人格と手順は別物
- **deploy は `.skills/` に住めない** — Webサービスと作業手順は別物
- **ontology は `.deploy/` に住めない** — 定義と実行は別物
- **crx は `.extension/` に住めない** — Chrome拡張とPi拡張は別物

## 引っ越し規定

1. **同 type 内の引っ越し**は自由（`.skills/dev/` → `.skills/observe/`）
2. **異 type への引っ越し**は、type 変更とみなし、新しい戸籍番号を発行する
3. **名札は引っ越しとともに移動**し、`moved_from` に旧パスを記録
4. **台帳も同時に更新**し、弁慶が同期を確認する

## 監視

弁慶は、台帳同期時に `path` と `type` の組み合わせを検証し、違反があれば警告を発する。
