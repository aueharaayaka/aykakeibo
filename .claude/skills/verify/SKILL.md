---
name: verify
description: aykakeibo（単一ファイルPWA）の変更をブラウザで実際に動かして検証する手順
---

# aykakeibo の動作検証

ビルド不要の静的アプリ。HTTPサーバで配信して Playwright (chromium) で駆動する。

## 起動

```bash
python3 -m http.server 8901 --directory /path/to/aykakeibo &
```

リモート環境では `npm install playwright-core`（scratchpad内でよい）し、
`executablePath` に `/opt/pw-browsers/chromium-1194/chrome-linux/chrome`
（バージョン付きディレクトリ。`/opt/pw-browsers/` を `ls` して確認）を渡す。
ビューポートは 430x932（モバイル前提レイアウト）。

## データ投入

`page.addInitScript()` で `localStorage.setItem('kakeibo_state', JSON.stringify(state))`
してから `goto` する。state の形は CLAUDE.md の State Data Model を参照。

## 注意点（ハマりどころ）

- 支出の `date` は `'M/D'` 形式（例 `'7/6'`、ゼロ埋めなし・年なし）。
  年は月バケットキー `'YYYY-MM'` から補完する。CLAUDE.md 旧版の
  `'YYYY-MM-DD'` 記載は誤りだった。
- 画面遷移は `.nav-btn:has-text("設定")` などナビボタンのクリックで行える。
- 起動時に祝日APIへの fetch が走る。オフラインでも console に404等が出る
  ことがあるが、ハードコードの `JAPAN_HOLIDAYS` にフォールバックするので無害。
- 月次の主要表示は `recalc()` が更新する。支出の追加/削除は `renderAll()`
  ではなく `recalc()` を呼ぶ経路なので、表示更新の検証は支出追加後の値を見る。

## 検証しがいのあるフロー

- 支出登録（用途セレクト → 金額 → 登録ボタン）→ 食費合計・残高の連動
- 月またぎ（前月データを `'YYYY-MM'` バケットに入れて挙動確認）
- 初回起動（localStorage空）でエラーが出ないこと（`pageerror` を監視）
