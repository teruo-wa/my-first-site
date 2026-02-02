# GAS固有の制約と対策

Google Apps Script Webアプリ開発における技術的制約と、HIG準拠のための代替実装パターン。

---

## 0. スタンドアロンWebアプリの注意点

### getActiveSpreadsheet() は使えない

スタンドアロンのWebアプリでは `SpreadsheetApp.getActiveSpreadsheet()` は `null` を返す。
必ず `PropertiesService` でスプレッドシートIDを管理すること。

```javascript
// NG: スタンドアロンでは動作しない
function getData() {
  const ss = SpreadsheetApp.getActiveSpreadsheet(); // null!
  // ...
}

// OK: スタンドアロンでも動作する
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
```

### シート名は定数で管理

ハードコードを避け、保守性を高める。

```javascript
const SHEET_NAMES = {
  DATA: 'データ',
  COMMENTS: 'コメント',
  SETTINGS: '設定',
  ARCHIVE: 'アーカイブ'
};

// 使用例
const sheet = ss.getSheetByName(SHEET_NAMES.DATA);
```

---

## 1. 非同期処理と待機時間

### 制約

GAS関数は `google.script.run` を介してサーバーで実行されるため、**1-3秒の遅延**が発生。
0.1秒以内の瞬時反応は不可能。

### 対応策

1. すべてのGAS呼び出しにローディング表示
2. 処理完了後に明確なフィードバック（トースト通知等）
3. 連続クリック防止のためボタンをdisabled

```javascript
function callGASFunction() {
  const btn = document.getElementById('actionBtn');
  btn.disabled = true;
  btn.innerHTML = '<span class="spinner"></span> 処理中...';

  google.script.run
    .withSuccessHandler((result) => {
      btn.disabled = false;
      btn.innerHTML = '実行';
      showToast('完了しました', 'success');
      updateUI(result);
    })
    .withFailureHandler((error) => {
      btn.disabled = false;
      btn.innerHTML = '実行';
      showError('エラーが発生しました: ' + error.message);
    })
    .gasFunction();
}
```

### スピナーCSS

```css
.spinner {
  display: inline-block;
  width: 16px;
  height: 16px;
  border: 2px solid transparent;
  border-top-color: currentColor;
  border-radius: 50%;
  animation: spin 0.8s linear infinite;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}
```

---

## 2. 処理の中断不可

### 制約

GAS関数実行中は処理を中断できない。`google.script.run` で呼び出した関数はサーバー側で完了まで実行される。

### 対応策

クライアント側でキャンセルフラグを立て、結果を破棄する。

```javascript
let isCancelled = false;

function executeWithCancel() {
  isCancelled = false;
  showProgress('処理を開始しています...');

  google.script.run
    .withSuccessHandler((result) => {
      if (!isCancelled) {
        applyResult(result);
        showToast('完了しました', 'success');
      } else {
        showToast('処理は完了しましたが結果は破棄されました', 'info');
      }
      hideProgress();
    })
    .longRunningFunction();
}

function cancelOperation() {
  isCancelled = true;
  showToast('キャンセルしました（サーバー処理は完了まで続行されます）', 'warning');
}
```

---

## 3. 取り消し不可能な操作

### 制約

以下の操作は技術的に取り消せない：
- スプレッドシートの編集
- ファイル削除
- メール送信
- カレンダー予定の作成/削除

### 対応策（優先順）

#### 1. 論理削除（最優先）

削除フラグやアーカイブ列を使用。

```javascript
// クライアント側
function deleteItem(id) {
  google.script.run
    .withSuccessHandler(() => {
      showToast('アーカイブしました', 'success', {
        action: '元に戻す',
        callback: () => restoreItem(id)
      });
      refreshList();
    })
    .archiveItem(id);
}

function restoreItem(id) {
  google.script.run
    .withSuccessHandler(() => {
      showToast('復元しました', 'success');
      refreshList();
    })
    .restoreItem(id);
}
```

```javascript
// GAS側（コード.gs）
function archiveItem(id) {
  const sheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName('データ');
  const data = sheet.getDataRange().getValues();
  const headers = data[0];
  const statusCol = headers.indexOf('ステータス') + 1;
  const archivedAtCol = headers.indexOf('アーカイブ日時') + 1;

  for (let i = 1; i < data.length; i++) {
    if (data[i][0] == id) {
      sheet.getRange(i + 1, statusCol).setValue('ARCHIVED');
      sheet.getRange(i + 1, archivedAtCol).setValue(new Date());
      break;
    }
  }

  return { success: true };
}

function restoreItem(id) {
  const sheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName('データ');
  const data = sheet.getDataRange().getValues();
  const headers = data[0];
  const statusCol = headers.indexOf('ステータス') + 1;

  for (let i = 1; i < data.length; i++) {
    if (data[i][0] == id) {
      sheet.getRange(i + 1, statusCol).setValue('ACTIVE');
      break;
    }
  }

  return { success: true };
}
```

#### 2. 確認ダイアログ（次善）

不可逆的操作には「取り消せません」を明記。

```javascript
function permanentlyDelete(id) {
  if (confirm('完全に削除しますか？\n\nこの操作は取り消せません。\nアーカイブではなく完全削除されます。')) {
    google.script.run
      .withSuccessHandler(() => {
        showToast('削除しました', 'success');
        refreshList();
      })
      .permanentlyDeleteItem(id);
  }
}
```

#### 3. バックアップ機能（理想）

操作前に自動バックアップを作成。

```javascript
// GAS側
function createBackup() {
  const ss = SpreadsheetApp.getActiveSpreadsheet();
  const backupFolder = DriveApp.getFolderById('BACKUP_FOLDER_ID');
  const backupName = `${ss.getName()}_backup_${new Date().toISOString()}`;
  ss.copy(backupName).moveTo(backupFolder);
}
```

---

## 4. リアルタイム性の限界

### 制約

WebSocketやServer-Sent Eventsが使えないため、リアルタイム更新は困難。

### 対応策

#### ポーリング（一定間隔での再取得）

```javascript
let pollingInterval;

function startPolling(intervalMs = 30000) {
  pollingInterval = setInterval(() => {
    google.script.run
      .withSuccessHandler(updateData)
      .getData();
  }, intervalMs);
}

function stopPolling() {
  clearInterval(pollingInterval);
}

// ページ離脱時に停止
window.addEventListener('beforeunload', stopPolling);
```

#### 手動更新ボタン

```html
<button onclick="refreshData()" title="更新">
  <span class="material-icons">refresh</span>
</button>
<span class="last-updated">最終更新: <span id="lastUpdated">--:--</span></span>
```

```javascript
function refreshData() {
  const refreshBtn = document.querySelector('[title="更新"]');
  refreshBtn.disabled = true;
  refreshBtn.classList.add('spinning');

  google.script.run
    .withSuccessHandler((data) => {
      updateData(data);
      document.getElementById('lastUpdated').textContent =
        new Date().toLocaleTimeString('ja-JP');
      refreshBtn.disabled = false;
      refreshBtn.classList.remove('spinning');
    })
    .getData();
}
```

---

## 5. セッション管理の制約

### 制約

GASには標準的なセッション管理機能がない。

### 対応策

#### LocalStorage（クライアント側）

```javascript
// 設定の保存
function saveSettings(settings) {
  localStorage.setItem('appSettings', JSON.stringify(settings));
}

// 設定の読み込み
function loadSettings() {
  const saved = localStorage.getItem('appSettings');
  return saved ? JSON.parse(saved) : getDefaultSettings();
}
```

#### PropertiesService（サーバー側）

```javascript
// GAS側
function saveUserPreference(key, value) {
  const userProperties = PropertiesService.getUserProperties();
  userProperties.setProperty(key, JSON.stringify(value));
}

function getUserPreference(key) {
  const userProperties = PropertiesService.getUserProperties();
  const value = userProperties.getProperty(key);
  return value ? JSON.parse(value) : null;
}
```

---

## 6. 入力サジェストの遅延

### 制約

動的に候補を取得する場合、1-2秒の遅延が発生。

### 対応策

静的datalistを優先。動的取得が必要な場合はデバウンス（500ms）。

```javascript
let debounceTimer;
const input = document.getElementById('searchInput');
const suggestionsDiv = document.getElementById('suggestions');

input.addEventListener('input', () => {
  clearTimeout(debounceTimer);

  const query = input.value.trim();
  if (query.length < 2) {
    suggestionsDiv.innerHTML = '';
    return;
  }

  // ローディング表示
  suggestionsDiv.innerHTML = '<div class="loading">候補を検索中...</div>';

  // デバウンス: 500ms待ってから検索
  debounceTimer = setTimeout(() => {
    google.script.run
      .withSuccessHandler((suggestions) => {
        displaySuggestions(suggestions);
      })
      .withFailureHandler(() => {
        suggestionsDiv.innerHTML = '';
      })
      .getSuggestions(query);
  }, 500);
});

function displaySuggestions(suggestions) {
  if (suggestions.length === 0) {
    suggestionsDiv.innerHTML = '<div class="no-results">候補が見つかりません</div>';
    return;
  }

  suggestionsDiv.innerHTML = suggestions
    .map(s => `<div class="suggestion-item" onclick="selectSuggestion('${s}')">${s}</div>`)
    .join('');
}
```

---

---

## 7. エラーハンドリングのベストプラクティス

### GAS側: 必ずtry-catchで囲む

```javascript
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

### 返却値の統一フォーマット

```javascript
// 成功時
return { success: true, data: result };

// 失敗時
return { success: false, error: 'エラーメッセージ' };
```

### クライアント側: 両方のハンドラを必ず実装

```javascript
google.script.run
  .withSuccessHandler(result => {
    if (result.success) {
      // 成功処理
      updateUI(result.data);
    } else {
      // GAS側で捕捉したエラー
      showToast(result.error, 'danger');
    }
  })
  .withFailureHandler(error => {
    // GAS実行自体の失敗（ネットワークエラー等）
    showToast('通信エラー: ' + error.message, 'danger');
  })
  .getData();
```

### Logger.log vs console.error

```javascript
// デバッグ用（GASエディタの「実行ログ」で確認）
Logger.log('Debug: ' + JSON.stringify(data));

// エラー用（Stackdriver Loggingに記録）
console.error('Error:', e);
```

**注意**: Webアプリのクライアント側では `console.log` が使えるが、GAS側の `console.log` はWebアプリからは見えない。

---

## トースト通知の実装

GAS制約対応で頻繁に使用するトースト通知の実装例。

```javascript
function showToast(message, type = 'info', options = {}) {
  const toast = document.createElement('div');
  toast.className = `toast toast--${type}`;
  toast.innerHTML = `
    <span class="toast__message">${message}</span>
    ${options.action ? `<button class="toast__action" onclick="(${options.callback})();this.parentElement.remove();">${options.action}</button>` : ''}
  `;

  document.body.appendChild(toast);

  // 自動削除
  setTimeout(() => toast.remove(), options.duration || 3000);
}
```

```css
.toast {
  position: fixed;
  bottom: var(--space-4);
  left: 50%;
  transform: translateX(-50%);
  padding: var(--space-4) var(--space-6);
  border-radius: var(--radius-md);
  background: var(--bg-surface);
  color: var(--text-primary);
  box-shadow: var(--shadow);
  display: flex;
  align-items: center;
  gap: var(--space-4);
  z-index: 2000;
  animation: slideUp 0.3s ease;
}

.toast--success { border-left: 4px solid var(--color-success); }
.toast--warning { border-left: 4px solid var(--color-warning); }
.toast--danger { border-left: 4px solid var(--color-danger); }

@keyframes slideUp {
  from { transform: translateX(-50%) translateY(100%); opacity: 0; }
  to { transform: translateX(-50%) translateY(0); opacity: 1; }
}
```
