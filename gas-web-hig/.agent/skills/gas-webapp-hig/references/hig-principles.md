# HIG 20原則 詳細ガイド

Human Interface Guidelines（HIG）に基づくUI/UX原則の詳細解説とコード例。

---

## Phase 1: 全アプリ必須の基本原則（7項目）

### 1. ユーザーの主導権を保証する

システムがユーザーを一方的にコントロールしない。すべての操作にキャンセル機能を提供する。

**GAS制約**: 関数実行中は技術的に中断不可。クライアント側でキャンセルフラグを立て、結果を受け取らない形で実装。

```html
<div class="modal-actions">
  <button class="btn btn--ghost" onclick="closeModal()">キャンセル</button>
  <button class="btn btn--primary" onclick="saveData()">保存</button>
</div>
```

```javascript
let isCancelled = false;

function executeWithCancel() {
  isCancelled = false;
  showLoadingSpinner();

  google.script.run
    .withSuccessHandler((result) => {
      if (!isCancelled) {
        displayResult(result);
      }
      hideLoadingSpinner();
    })
    .longRunningFunction();
}

function cancelOperation() {
  isCancelled = true;
  hideLoadingSpinner();
  showToast('処理をキャンセルしました', 'info');
}
```

---

### 2. コンストレイント（制約）を活用する

現在実行できない操作はボタンをdisableにする。無効な状態では視覚的にグレーアウト。

```javascript
const submitBtn = document.getElementById('submitBtn');
const inputField = document.getElementById('input');
inputField.addEventListener('input', () => {
  submitBtn.disabled = inputField.value.trim() === '';
});
```

---

### 3. オブジェクトは自身の状態を体現する

選択中の項目は視覚的に明示（ハイライト、チェックマーク等）。処理中は「読み込み中」を表示。

```css
.item.selected {
  background: var(--accent);
  color: var(--text-inverse);
  border: 2px solid var(--accent);
}
.item:not(.selected) {
  opacity: 0.7;
}
```

---

### 4. すべての操作可能な要素は意味を持つ

タスクに関係ない要素は非表示またはdisableに。「常に表示されているが押せない」状態を避ける。

```javascript
if (userHasPermission) {
  document.getElementById('adminPanel').style.display = 'block';
} else {
  document.getElementById('adminPanel').style.display = 'none';
}
```

---

### 5. デフォルトボタンには具体的な動詞を用いる

「OK」「はい」は禁止。「保存」「削除」「送信」など具体的な動詞を使用。

```html
<!-- NG -->
<button onclick="deleteItem()">OK</button>

<!-- OK -->
<button onclick="deleteItem()">削除</button>
```

---

### 6. エラー表示は建設的にする

- 何が起きたのかを具体的に説明
- どうすれば解決できるかを示す
- プログラマー向けのエラーコードは表示しない

```javascript
// NG
alert('Error: Invalid input');

// OK
showError('メールアドレスの形式が正しくありません。「example@domain.com」のように入力してください。');
```

---

### 7. 黙って実行する（不要な確認を排除）

可逆的な操作には確認ダイアログを出さない。正常完了のメッセージも基本不要（画面の変化で伝える）。

**例外**: データ削除など不可逆的な操作のみ確認する。

```javascript
// NG: 保存のたびに確認
if (confirm('保存しますか？')) { save(); }

// OK: 黙って保存、処理中と完了を明示
function save() {
  const saveBtn = document.getElementById('saveBtn');
  saveBtn.disabled = true;
  saveBtn.innerHTML = '<span class="spinner"></span> 保存中...';

  google.script.run
    .withSuccessHandler(() => {
      saveBtn.disabled = false;
      saveBtn.innerHTML = '保存';
      showToast('保存しました', 'success');
      refreshList();
    })
    .saveData(data);
}

// 例外: 削除は確認が必要
if (confirm('このデータを削除しますか？この操作は取り消せません。')) {
  deleteData();
}
```

---

## Phase 2: フォーム・入力UIの原則（8項目）

### 8. 入力フォームにはストーリー性を持たせる

関連する項目をグループ化（視覚ゲシュタルト）。入力順序を論理的に（基本→詳細）。

```html
<form>
  <section class="form-section">
    <h3>基本情報</h3>
    <input type="text" placeholder="名前" />
    <input type="email" placeholder="メールアドレス" />
  </section>

  <section class="form-section">
    <h3>詳細設定</h3>
    <select>...</select>
    <textarea>...</textarea>
  </section>
</form>
```

---

### 9. 操作の流れを作る（ボタン重力）

アクションボタンは流れの終点に配置。主要ボタンは右下または下中央。

```css
.form-actions {
  display: flex;
  justify-content: flex-end;
  gap: var(--space-4);
  margin-top: var(--space-6);
}
```

---

### 10. 選択肢の文言は肯定文にする

チェックボックスやラジオボタンのラベルは肯定文で。

```html
<!-- NG -->
<input type="checkbox" id="noEmail">
<label for="noEmail">メールを受信しない</label>

<!-- OK -->
<input type="checkbox" id="receiveEmail">
<label for="receiveEmail">メールを受信する</label>
```

---

### 11. 値を入力させるのではなく結果を選ばせる

数値入力よりスライダーやステッパーを優先。パラメータではなく、得られる結果を示す。

```html
<!-- NG: 抽象的な数値 -->
<input type="number" min="1" max="10" placeholder="品質レベル" />

<!-- OK: 結果で示す -->
<input type="range" min="1" max="3" />
<div class="quality-labels">
  <span>標準</span>
  <span>高品質</span>
  <span>最高品質</span>
</div>
```

---

### 12. ユーザーに厳密さを求めない

全角/半角を自動変換。ハイフンや空白の有無を許容。

```javascript
function normalizePhoneNumber(input) {
  return input.replace(/[０-９]/g, s => String.fromCharCode(s.charCodeAt(0) - 0xFEE0))
              .replace(/[-ー−]/g, '');
}
```

---

### 13. 入力サジェスチョンを提示する

オートコンプリート機能を実装。過去の入力履歴から候補を表示。

**GAS制約**: 動的候補取得は1-2秒遅延。静的datalistを優先、動的取得はデバウンス（500ms）。

```html
<!-- 静的リスト推奨 -->
<input type="text" list="suggestions" placeholder="項目を入力" />
<datalist id="suggestions">
  <option value="東京都">
  <option value="神奈川県">
</datalist>
```

```javascript
// 動的取得の場合
let debounceTimer;
input.addEventListener('input', () => {
  clearTimeout(debounceTimer);
  if (input.value.length < 2) return;

  suggestionsDiv.innerHTML = '<div class="loading">候補を検索中...</div>';

  debounceTimer = setTimeout(() => {
    google.script.run
      .withSuccessHandler(displaySuggestions)
      .getSuggestions(input.value);
  }, 500);
});
```

---

### 14. フェールセーフを優先する

すべての操作を取り消し可能に。

**GAS制約**: スプレッドシート編集、ファイル削除等は取り消し不可。以下の代替策：

1. **優先**: 論理削除（削除フラグ）やアーカイブ機能
2. **次善**: 削除前の確認ダイアログ（「取り消せません」を明記）
3. **理想**: バックアップやバージョン管理機能

```javascript
// 論理削除推奨
function deleteItem(id) {
  google.script.run
    .withSuccessHandler(() => {
      showToast('アーカイブしました', 'success', {
        action: '元に戻す',
        callback: () => restoreItem(id)
      });
    })
    .archiveItem(id);
}

// GAS側
function archiveItem(id) {
  const sheet = SpreadsheetApp.getActiveSheet();
  // 削除フラグ列に'ARCHIVED'を記録
}
```

---

### 15. 操作の近くでフィードバックする

エラーメッセージは該当の入力欄の近くに表示。

```html
<div class="input-group">
  <input type="email" id="email" class="error" />
  <span class="error-message">メールアドレスの形式が正しくありません</span>
</div>
```

```css
.error-message {
  position: absolute;
  top: 100%;
  color: var(--color-danger);
  font-size: var(--font-size-sm);
}
```

---

## Phase 3: UX向上の原則（5項目）

### 16. ユーザーの記憶に頼らない

必要な情報はその場で参照可能に。入力例をプレースホルダーで示す。

```html
<input type="text" placeholder="例: 03-1234-5678" />
<label>
  パスワード
  <span class="help-icon" title="8文字以上、英数字を含む">?</span>
</label>
```

---

### 17. データよりも情報を伝える

数値だけでなく意味を伝える。

```javascript
// NG: 123MB使用中
// OK: 残り80%

function formatStorage(usedBytes, totalBytes) {
  const percent = ((usedBytes / totalBytes) * 100).toFixed(0);
  return `使用中（残り${100-percent}%）`;
}
```

---

### 18. 即座の喜びを与える

初回起動時にサンプルデータを表示。空の状態（Empty State）を避ける。

```javascript
if (localStorage.getItem('isFirstVisit') === null) {
  loadSampleData();
  startTour();
  localStorage.setItem('isFirstVisit', 'false');
}
```

---

### 19. 回答の先送り（必須項目の最小化）

本当に必須な項目だけを必須に。

```html
<form>
  <input type="email" required placeholder="メールアドレス（必須）" />
  <input type="text" placeholder="電話番号（任意）" />
</form>
```

---

### 20. プログレッシブ・ディスクロージャ（段階的開示）

高度な機能は最初は隠す。「詳細設定」で折りたたむ。

```html
<details class="advanced-settings">
  <summary>詳細設定</summary>
  <div class="advanced-content">
    <!-- 高度な設定項目 -->
  </div>
</details>
```
