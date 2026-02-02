# Googleサービス連携パターン

GAS Webアプリで各種Googleサービスと連携するためのパターン集。
スプレッドシートに限らず、様々なサービスと連携可能。

---

## 1. スプレッドシート連携

### 基本パターン（スタンドアロン対応）

```javascript
const SHEET_NAMES = {
  DATA: 'データ',
  SETTINGS: '設定',
  ARCHIVE: 'アーカイブ'
};

// スプレッドシート自動作成（初回アクセス時）
function getOrCreateSpreadsheet() {
  const props = PropertiesService.getScriptProperties();
  let ssId = props.getProperty('SPREADSHEET_ID');

  if (!ssId) {
    const ss = SpreadsheetApp.create('アプリ名_データ');
    ssId = ss.getId();
    props.setProperty('SPREADSHEET_ID', ssId);
    initializeSheets(ss);
    Logger.log('Created new spreadsheet: ' + ssId);
  }

  return SpreadsheetApp.openById(ssId);
}

// シート初期化
function initializeSheets(ss) {
  // デフォルトのSheet1を削除
  const defaultSheet = ss.getSheetByName('シート1');

  // データシート作成
  const dataSheet = ss.insertSheet(SHEET_NAMES.DATA);
  dataSheet.appendRow(['ID', '名前', '作成日時', 'ステータス']);

  // 設定シート作成
  const settingsSheet = ss.insertSheet(SHEET_NAMES.SETTINGS);
  settingsSheet.appendRow(['キー', '値']);

  // デフォルトシート削除
  if (defaultSheet) {
    ss.deleteSheet(defaultSheet);
  }
}
```

### データ取得（エラーハンドリング付き）

```javascript
function getData() {
  try {
    const ss = getOrCreateSpreadsheet();
    const sheet = ss.getSheetByName(SHEET_NAMES.DATA);

    if (!sheet) {
      return { success: false, error: 'シートが見つかりません' };
    }

    const data = sheet.getDataRange().getValues();
    const headers = data[0];
    const rows = data.slice(1).filter(row => row[0]); // 空行除外

    // オブジェクト形式に変換
    const items = rows.map(row => {
      const item = {};
      headers.forEach((header, i) => {
        item[header] = row[i];
      });
      return item;
    });

    return { success: true, data: items };
  } catch (e) {
    console.error('getData error:', e);
    return { success: false, error: e.message };
  }
}
```

### データ保存

```javascript
function saveData(item) {
  try {
    const ss = getOrCreateSpreadsheet();
    const sheet = ss.getSheetByName(SHEET_NAMES.DATA);

    // ID生成
    const id = Utilities.getUuid();
    const timestamp = new Date();

    sheet.appendRow([
      id,
      item.name,
      timestamp,
      'ACTIVE'
    ]);

    return { success: true, id: id };
  } catch (e) {
    console.error('saveData error:', e);
    return { success: false, error: e.message };
  }
}
```

### 論理削除（アーカイブ）

```javascript
function archiveItem(id) {
  try {
    const ss = getOrCreateSpreadsheet();
    const sheet = ss.getSheetByName(SHEET_NAMES.DATA);
    const data = sheet.getDataRange().getValues();
    const headers = data[0];
    const statusCol = headers.indexOf('ステータス') + 1;

    for (let i = 1; i < data.length; i++) {
      if (data[i][0] === id) {
        sheet.getRange(i + 1, statusCol).setValue('ARCHIVED');
        return { success: true };
      }
    }

    return { success: false, error: 'データが見つかりません' };
  } catch (e) {
    console.error('archiveItem error:', e);
    return { success: false, error: e.message };
  }
}
```

---

## 2. Googleドキュメント連携

### ドキュメント作成

```javascript
function createDocument(title, content) {
  try {
    const doc = DocumentApp.create(title);
    const body = doc.getBody();

    // タイトル追加
    body.appendParagraph(title)
      .setHeading(DocumentApp.ParagraphHeading.HEADING1);

    // 本文追加
    body.appendParagraph(content);

    // 日付追加
    body.appendParagraph('作成日: ' + new Date().toLocaleDateString('ja-JP'));

    doc.saveAndClose();

    return {
      success: true,
      id: doc.getId(),
      url: doc.getUrl()
    };
  } catch (e) {
    console.error('createDocument error:', e);
    return { success: false, error: e.message };
  }
}
```

### ドキュメントからテキスト取得

```javascript
function getDocumentText(docId) {
  try {
    const doc = DocumentApp.openById(docId);
    const text = doc.getBody().getText();

    return { success: true, text: text };
  } catch (e) {
    console.error('getDocumentText error:', e);
    return { success: false, error: e.message };
  }
}
```

---

## 3. Googleスライド連携

### スライド作成

```javascript
function createPresentation(title, slides) {
  try {
    const presentation = SlidesApp.create(title);

    // 最初のスライド（タイトル）
    const titleSlide = presentation.getSlides()[0];
    titleSlide.getShapes()[0].getText().setText(title);

    // 追加スライド
    slides.forEach(slideData => {
      const slide = presentation.appendSlide(SlidesApp.PredefinedLayout.TITLE_AND_BODY);
      const shapes = slide.getShapes();

      if (shapes.length > 0) {
        shapes[0].getText().setText(slideData.title);
      }
      if (shapes.length > 1) {
        shapes[1].getText().setText(slideData.body);
      }
    });

    return {
      success: true,
      id: presentation.getId(),
      url: presentation.getUrl()
    };
  } catch (e) {
    console.error('createPresentation error:', e);
    return { success: false, error: e.message };
  }
}
```

---

## 4. Googleカレンダー連携

### 予定取得

```javascript
function getCalendarEvents(startDate, endDate) {
  try {
    const calendar = CalendarApp.getDefaultCalendar();
    const events = calendar.getEvents(new Date(startDate), new Date(endDate));

    const eventList = events.map(event => ({
      id: event.getId(),
      title: event.getTitle(),
      start: event.getStartTime(),
      end: event.getEndTime(),
      description: event.getDescription(),
      location: event.getLocation()
    }));

    return { success: true, events: eventList };
  } catch (e) {
    console.error('getCalendarEvents error:', e);
    return { success: false, error: e.message };
  }
}
```

### 予定作成

```javascript
function createCalendarEvent(eventData) {
  try {
    const calendar = CalendarApp.getDefaultCalendar();

    const event = calendar.createEvent(
      eventData.title,
      new Date(eventData.start),
      new Date(eventData.end),
      {
        description: eventData.description,
        location: eventData.location
      }
    );

    return {
      success: true,
      id: event.getId(),
      url: event.getUrl()
    };
  } catch (e) {
    console.error('createCalendarEvent error:', e);
    return { success: false, error: e.message };
  }
}
```

---

## 5. Gmail連携

### メール送信

```javascript
function sendEmail(to, subject, body, options = {}) {
  try {
    GmailApp.sendEmail(to, subject, body, {
      htmlBody: options.htmlBody,
      cc: options.cc,
      bcc: options.bcc,
      name: options.senderName || 'アプリ名',
      attachments: options.attachments
    });

    return { success: true };
  } catch (e) {
    console.error('sendEmail error:', e);
    return { success: false, error: e.message };
  }
}
```

### 下書き作成

```javascript
function createDraft(to, subject, body) {
  try {
    const draft = GmailApp.createDraft(to, subject, body);

    return {
      success: true,
      id: draft.getId()
    };
  } catch (e) {
    console.error('createDraft error:', e);
    return { success: false, error: e.message };
  }
}
```

---

## 6. Google Chat連携

### Webhook経由でメッセージ送信

```javascript
function sendChatMessage(webhookUrl, message) {
  try {
    const payload = {
      text: message
    };

    const options = {
      method: 'post',
      contentType: 'application/json',
      payload: JSON.stringify(payload)
    };

    UrlFetchApp.fetch(webhookUrl, options);

    return { success: true };
  } catch (e) {
    console.error('sendChatMessage error:', e);
    return { success: false, error: e.message };
  }
}
```

### カード形式のメッセージ

```javascript
function sendChatCard(webhookUrl, title, subtitle, sections) {
  try {
    const payload = {
      cards: [{
        header: {
          title: title,
          subtitle: subtitle
        },
        sections: sections.map(section => ({
          header: section.header,
          widgets: section.widgets
        }))
      }]
    };

    const options = {
      method: 'post',
      contentType: 'application/json',
      payload: JSON.stringify(payload)
    };

    UrlFetchApp.fetch(webhookUrl, options);

    return { success: true };
  } catch (e) {
    console.error('sendChatCard error:', e);
    return { success: false, error: e.message };
  }
}
```

---

## 7. Googleドライブ連携

### フォルダ作成

```javascript
function createFolder(folderName, parentFolderId = null) {
  try {
    let folder;

    if (parentFolderId) {
      const parentFolder = DriveApp.getFolderById(parentFolderId);
      folder = parentFolder.createFolder(folderName);
    } else {
      folder = DriveApp.createFolder(folderName);
    }

    return {
      success: true,
      id: folder.getId(),
      url: folder.getUrl()
    };
  } catch (e) {
    console.error('createFolder error:', e);
    return { success: false, error: e.message };
  }
}
```

### ファイルアップロード

```javascript
function uploadFile(base64Data, fileName, mimeType, folderId = null) {
  try {
    const blob = Utilities.newBlob(
      Utilities.base64Decode(base64Data),
      mimeType,
      fileName
    );

    let file;
    if (folderId) {
      const folder = DriveApp.getFolderById(folderId);
      file = folder.createFile(blob);
    } else {
      file = DriveApp.createFile(blob);
    }

    return {
      success: true,
      id: file.getId(),
      url: file.getUrl()
    };
  } catch (e) {
    console.error('uploadFile error:', e);
    return { success: false, error: e.message };
  }
}
```

---

## appsscript.json のスコープ設定

使用するサービスに応じてスコープを設定：

```json
{
  "timeZone": "Asia/Tokyo",
  "dependencies": {},
  "exceptionLogging": "STACKDRIVER",
  "runtimeVersion": "V8",
  "webapp": {
    "executeAs": "USER_DEPLOYING",
    "access": "ANYONE_ANONYMOUS"
  },
  "oauthScopes": [
    "https://www.googleapis.com/auth/spreadsheets",
    "https://www.googleapis.com/auth/documents",
    "https://www.googleapis.com/auth/presentations",
    "https://www.googleapis.com/auth/calendar",
    "https://www.googleapis.com/auth/gmail.send",
    "https://www.googleapis.com/auth/drive"
  ]
}
```

**注意**: 必要なスコープのみ含めること。スコープが多いと承認画面でユーザーに警告が表示される。
