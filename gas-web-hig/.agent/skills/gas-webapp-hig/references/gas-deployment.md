# GAS Webアプリのデプロイ手順

GAS WebアプリはGASエディタまたはclaspを使ってデプロイできる。
**重要な注意点**: claspとGASエディタでの挙動の違いを理解すること。

---

## 1. GASエディタでのデプロイ（推奨）

### 初回デプロイ

1. GASエディタで「デプロイ」→「新しいデプロイ」をクリック
2. 種類：「ウェブアプリ」を選択
3. 説明：バージョン説明を入力（例：「v1.0 初回リリース」）
4. 実行ユーザー：「自分」を選択
5. アクセスできるユーザー：
   - 「全員」: 誰でもアクセス可能（認証不要）
   - 「全員（Googleアカウント必須）」: ログイン必須
   - 「自分のみ」: テスト用
6. 「デプロイ」をクリック
7. 表示されたURLをコピー

### コード更新後の再デプロイ

**重要**: コードを編集した後は必ず「新しいデプロイ」を作成する必要がある。

| 方法 | 説明 | URL |
|-----|------|-----|
| 新しいデプロイ | 本番用。毎回URLが変わる | `https://script.google.com/macros/s/xxx/exec` |
| テストデプロイ | 開発用。URLが固定 | `https://script.google.com/macros/s/xxx/dev` |

開発中は「テストデプロイ」を使うとURLが変わらず便利。

---

## 2. claspでのデプロイ

### 前提条件

```bash
npm install -g @google/clasp
clasp login
```

### プロジェクト設定

```bash
# 既存プロジェクトをクローン
clasp clone <SCRIPT_ID>

# または新規作成
clasp create --title "アプリ名" --type webapp
```

### .clasp.json の設定

```json
{
  "scriptId": "YOUR_SCRIPT_ID",
  "rootDir": "",
  "scriptExtensions": [".js", ".gs"],
  "htmlExtensions": [".html"],
  "jsonExtensions": [".json"]
}
```

### appsscript.json の設定（必須）

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

**注意**: `webapp` セクションがないとWebアプリとしてデプロイできない。

### コードのプッシュ

```bash
clasp push
```

### 重要な注意点

**`clasp push` はGASエディタにコードを反映するだけ**

- Webアプリの公開URLには反映されない
- Webアプリに反映するには「新しいデプロイ」が必要
- GASエディタで「デプロイ」→「新しいデプロイ」を実行する

### clasp deployコマンド

```bash
# 新しいデプロイを作成
clasp deploy --description "v1.0"

# デプロイ一覧を確認
clasp deployments

# 特定のデプロイを更新
clasp deploy --deploymentId <DEPLOYMENT_ID> --description "v1.1"
```

---

## 3. デバッグのヒント

### 問題: Webアプリでデータが表示されない

1. **GASエディタで直接関数を実行**
   - 関数名を選択して「実行」ボタンをクリック
   - 「実行ログ」で結果を確認

2. **ハードコードしたテストデータを返してみる**
   ```javascript
   function getData() {
     // テスト: 固定値を返す
     return { success: true, data: [['テスト太郎', 'テストクラス']] };
   }
   ```
   - これで表示されれば、デプロイの問題
   - 表示されなければ、クライアント側の問題

3. **Logger.log を使う**
   ```javascript
   function getData() {
     Logger.log('getData called');
     const ss = getOrCreateSpreadsheet();
     Logger.log('Spreadsheet ID: ' + ss.getId());
     // ...
   }
   ```
   - GASエディタの「実行ログ」で確認
   - **注意**: console.log はWebアプリでは見えない

4. **新しいデプロイを作成**
   - clasp push 後は必ず「新しいデプロイ」を作成

### 問題: 「承認が必要です」エラー

1. GASエディタで任意の関数を実行
2. 承認ダイアログで「詳細」→「安全でないページに移動」
3. 権限を許可

### 問題: CORS エラー

GAS Webアプリは iframe 内で実行されるため、一部の外部リソースにアクセスできない。
解決策：

- 外部API呼び出しはサーバーサイド（Code.gs）で行う
- `UrlFetchApp.fetch()` を使用

---

## 4. 本番運用時のベストプラクティス

### バージョン管理

```
v1.0.0 - 初回リリース
v1.0.1 - バグ修正
v1.1.0 - 新機能追加
v2.0.0 - 破壊的変更
```

### デプロイ前チェックリスト

- [ ] GASエディタで全関数が正常に動作するか確認
- [ ] テストデプロイで動作確認
- [ ] appsscript.json の設定が正しいか確認
- [ ] 必要なスコープが含まれているか確認
- [ ] エラーハンドリングが実装されているか確認

### ロールバック

問題が発生した場合：

1. GASエディタで「デプロイを管理」をクリック
2. 以前のバージョンを選択
3. 「デプロイ」をクリック
