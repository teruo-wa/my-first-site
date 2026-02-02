---
name: gas-webapp-hig
description: |
  GAS Webアプリ開発スキル（HIG準拠版）。
  Use when: GASでWebアプリを作成、doGet、スプレッドシート連携Webアプリ、HIG準拠、GASウェブアプリ、gas-webapp、Apps Script Webアプリ、Googleサービス連携、Gemini API連携の時。
---

# GAS Webアプリ開発スキル（HIG準拠版）

Google Apps Script（GAS）で高品質なWebアプリを開発するためのスキル。
Apple Human Interface Guidelines（HIG）の20原則を組み込み、業界標準のUI/UXを保証する。

## このスキルを使用する時

- GASでWebアプリを作成したい
- `doGet()`でHTMLを返すアプリを開発
- Googleサービス連携のWebアプリ（スプレッドシート、ドキュメント、スライド、カレンダー、Gmail、Chat等）
- Gemini APIなど外部AI連携のWebアプリ
- 高品質なUI/UXのGASアプリが必要

## このスキルを使用しない時

- スタンドアロンスクリプト（トリガーのみで動作、UI不要）
- Gmailフィルタリング・自動返信（バックグラウンド処理のみ）
- スプレッドシートのカスタム関数（CUSTOM_FUNCTION）
- Apps Script APIの利用（外部からGASを呼び出す）

---

## 核心ルール

### 1. 言語規則（厳守）

**UIテキストは全て日本語またはカタカナのみ**

```
NG: Settings, Home, New, Light, Dark
OK: 設定, ホーム, 新規作成, ライト, ダーク
```

例外: フッター署名部分のみ英語可

### 2. テーマシステム（6種必須）

| テーマ名 | data-theme値 |
|---------|-------------|
| ライト | light |
| ダーク | dark |
| オーシャン | ocean |
| フォレスト | forest |
| サンセット | sunset |
| サクラ | sakura |

### 3. 必須機能

1. **ロード画面**: テーマ色同期、フェードアウト
2. **ツアー機能**: Driver.js v1.0.1、最低5ステップ
3. **設定モーダル**: テーマ選択、フォント設定、リセット
4. **LocalStorage**: 設定の永続化

### 4. GAS固有の制約対応

| 制約 | 対応策 |
|-----|-------|
| 1-3秒の遅延 | ローディング表示 + ボタンdisabled |
| 処理中断不可 | クライアント側キャンセルフラグ |
| 取り消し不可 | 論理削除（アーカイブ）優先 |
| リアルタイム不可 | ポーリング or 手動更新 |

---

## HIG 20原則（概要）

詳細は `references/hig-principles.md` を参照。

### Phase 1: 全アプリ必須（7項目）

1. ユーザーの主導権を保証する
2. コンストレイント（制約）を活用する
3. オブジェクトは自身の状態を体現する
4. すべての操作可能な要素は意味を持つ
5. デフォルトボタンには具体的な動詞を用いる
6. エラー表示は建設的にする
7. 黙って実行する（不要な確認を排除）

### Phase 2: フォーム・入力UI（8項目）

8. 入力フォームにはストーリー性を持たせる
9. 操作の流れを作る（ボタン重力）
10. 選択肢の文言は肯定文にする
11. 値を入力させるのではなく結果を選ばせる
12. ユーザーに厳密さを求めない
13. 入力サジェスチョンを提示する
14. フェールセーフを優先する
15. 操作の近くでフィードバックする

### Phase 3: UX向上（5項目）

16. ユーザーの記憶に頼らない
17. データよりも情報を伝える
18. 即座の喜びを与える
19. 回答の先送り（必須項目の最小化）
20. プログレッシブ・ディスクロージャ

---

## ワークフロー

### Step 1: 要件確認

以下をヒアリング：

- アプリ名（日本語/カタカナ）
- 目的・機能
- 対象ユーザー
- **連携するGoogleサービス**（スプレッドシート、ドキュメント、スライド、カレンダー、Gmail、Chat等）
- **外部API連携の有無**（Gemini API等）
- 必要なCRUD操作

### Step 2: 設計

1. データ構造（連携サービスに応じた設計）
2. 画面構成（ヘッダー、メイン、モーダル）
3. GAS関数一覧
4. **appsscript.json の設定**（OAuth スコープ等）

### Step 3: 実装

`references/design-system.md` のテンプレートを使用。

出力ファイル：
- **Code.gs**: doGet関数 + サーバーサイド処理
- **Index.html**: HTML/CSS/JS統合ファイル
- **appsscript.json**: プロジェクト設定（必須）

### Step 4: デプロイ

`references/gas-deployment.md` の手順に従ってデプロイ。

**重要**: clasp push だけではWebアプリに反映されない。必ず「新しいデプロイ」を作成すること。

### Step 5: チェックリスト確認

`references/checklist.md` の全項目を確認。

---

## 出力ファイル構成

### appsscript.json（必須）

```json
{
  "timeZone": "Asia/Tokyo",
  "dependencies": {},
  "exceptionLogging": "STACKDRIVER",
  "runtimeVersion": "V8",
  "webapp": {
    "executeAs": "USER_DEPLOYING",
    "access": "ANYONE_ANONYMOUS"
  }
}
```

### Code.gs

```javascript
// シート名定数（ハードコード回避）
const SHEET_NAMES = {
  DATA: 'データ',
  SETTINGS: '設定'
};

function doGet() {
  return HtmlService.createHtmlOutputFromFile('Index')
    .setTitle('アプリ名')
    .setXFrameOptionsMode(HtmlService.XFrameOptionsMode.ALLOWALL);
}

// スプレッドシート自動作成パターン（スタンドアロン対応）
function getOrCreateSpreadsheet() {
  const props = PropertiesService.getScriptProperties();
  let ssId = props.getProperty('SPREADSHEET_ID');

  if (!ssId) {
    const ss = SpreadsheetApp.create('アプリ名_データ');
    ssId = ss.getId();
    props.setProperty('SPREADSHEET_ID', ssId);
    initializeSheets(ss);
  }

  return SpreadsheetApp.openById(ssId);
}

// エラーハンドリング付きデータ取得
function getData() {
  try {
    const ss = getOrCreateSpreadsheet();
    const sheet = ss.getSheetByName(SHEET_NAMES.DATA);
    if (!sheet) {
      return { success: false, error: 'シートが見つかりません' };
    }
    const data = sheet.getDataRange().getValues();
    return { success: true, data: data };
  } catch (e) {
    console.error('getData error:', e);
    return { success: false, error: e.message };
  }
}
```

### Index.html

HTML, CSS, JavaScript を1ファイルに統合。
詳細テンプレートは `references/design-system.md` 参照。

---

## 詳細リファレンス

| ファイル | 内容 |
|---------|------|
| `references/hig-principles.md` | HIG 20原則の詳細とコード例 |
| `references/design-system.md` | CSS変数、テーマ、HTMLテンプレート |
| `references/gas-constraints.md` | GAS制約と対策の詳細 |
| `references/gas-deployment.md` | デプロイ手順（clasp/GASエディタ） |
| `references/gas-integrations.md` | Googleサービス連携パターン |
| `references/gas-ai-integration.md` | Gemini API連携 |
| `references/checklist.md` | 出力前セルフチェックリスト |

---

## クレジット

このスキルは以下のプロンプトをベースに作成されています：

**Original**: [gas-webapp-prompt-hig.md](https://github.com/akari-iku/gas-webapp-prompt/blob/main/prompt/gas-webapp-prompt-hig.md)
**Author**: Akari ([akari-iku](https://github.com/akari-iku))
**License**: MIT License

スキル化にあたり、500行ルールに従いreferencesに詳細を分離しました。
