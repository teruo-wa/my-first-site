# Gemini API連携ガイド

GAS WebアプリからGemini APIを利用するためのガイド。
テキスト生成、画像生成、マルチモーダル対応のパターンを解説。

---

## 重要: APIモデル名について

**Gemini APIのモデル名は頻繁に更新される。**

実装時は必ず公式ドキュメントで最新のモデル名を確認すること：
- [Google AI Studio](https://aistudio.google.com/)
- [Gemini API ドキュメント](https://ai.google.dev/gemini-api/docs)

### モデル名の例（2025年1月時点）

| 用途 | モデル名（例） | 備考 |
|-----|--------------|------|
| テキスト生成 | `gemini-2.0-flash` | 高速・低コスト |
| テキスト生成（高品質） | `gemini-2.0-pro` | 高品質・高コスト |
| 画像生成 | `gemini-2.0-flash-exp-image-generation` | 実験的機能 |
| マルチモーダル | `gemini-2.0-flash` | 画像入力対応 |

**注意**: モデル名は変更される可能性があるため、APIエラーが発生した場合は公式ドキュメントで最新のモデル名を確認すること。

---

## 1. APIキーの管理

### 方法1: PropertiesService（推奨）

サーバーサイドで安全に保存。

```javascript
// APIキーを保存
function setApiKey(apiKey) {
  const props = PropertiesService.getScriptProperties();
  props.setProperty('GEMINI_API_KEY', apiKey);
  return { success: true };
}

// APIキーを取得
function getApiKey() {
  const props = PropertiesService.getScriptProperties();
  return props.getProperty('GEMINI_API_KEY');
}

// APIキーの存在確認
function hasApiKey() {
  const apiKey = getApiKey();
  return { hasKey: !!apiKey };
}
```

### 方法2: 設定モーダルから入力

ユーザーがUIからAPIキーを入力し、サーバーに保存。

```html
<!-- Index.html -->
<div class="setting-item">
  <label>Gemini APIキー</label>
  <input type="password" id="apiKeyInput" placeholder="APIキーを入力">
  <button onclick="saveApiKey()">保存</button>
</div>

<script>
function saveApiKey() {
  const apiKey = document.getElementById('apiKeyInput').value;
  if (!apiKey) {
    showToast('APIキーを入力してください', 'warning');
    return;
  }

  google.script.run
    .withSuccessHandler(() => {
      showToast('APIキーを保存しました', 'success');
      document.getElementById('apiKeyInput').value = '';
    })
    .withFailureHandler(e => {
      showToast('保存に失敗しました: ' + e.message, 'danger');
    })
    .setApiKey(apiKey);
}
</script>
```

---

## 2. テキスト生成

### 基本パターン

```javascript
function generateText(prompt) {
  const apiKey = getApiKey();
  if (!apiKey) {
    return { success: false, error: 'APIキーが設定されていません' };
  }

  // ※モデル名は公式ドキュメントで最新を確認
  const model = 'gemini-2.0-flash';
  const url = `https://generativelanguage.googleapis.com/v1beta/models/${model}:generateContent?key=${apiKey}`;

  const payload = {
    contents: [{
      parts: [{ text: prompt }]
    }],
    generationConfig: {
      temperature: 0.7,
      maxOutputTokens: 2048
    }
  };

  try {
    const response = UrlFetchApp.fetch(url, {
      method: 'post',
      contentType: 'application/json',
      payload: JSON.stringify(payload),
      muteHttpExceptions: true
    });

    const result = JSON.parse(response.getContentText());

    if (result.error) {
      console.error('Gemini API error:', result.error);
      return { success: false, error: result.error.message };
    }

    const text = result.candidates?.[0]?.content?.parts?.[0]?.text;
    if (!text) {
      return { success: false, error: 'レスポンスにテキストが含まれていません' };
    }

    return { success: true, text: text };
  } catch (e) {
    console.error('generateText error:', e);
    return { success: false, error: e.message };
  }
}
```

### システムプロンプト付き

```javascript
function generateTextWithSystem(systemPrompt, userPrompt) {
  const apiKey = getApiKey();
  if (!apiKey) {
    return { success: false, error: 'APIキーが設定されていません' };
  }

  const model = 'gemini-2.0-flash';
  const url = `https://generativelanguage.googleapis.com/v1beta/models/${model}:generateContent?key=${apiKey}`;

  const payload = {
    contents: [{
      parts: [{ text: userPrompt }]
    }],
    systemInstruction: {
      parts: [{ text: systemPrompt }]
    },
    generationConfig: {
      temperature: 0.7,
      maxOutputTokens: 2048
    }
  };

  try {
    const response = UrlFetchApp.fetch(url, {
      method: 'post',
      contentType: 'application/json',
      payload: JSON.stringify(payload),
      muteHttpExceptions: true
    });

    const result = JSON.parse(response.getContentText());

    if (result.error) {
      return { success: false, error: result.error.message };
    }

    const text = result.candidates?.[0]?.content?.parts?.[0]?.text;
    return { success: true, text: text };
  } catch (e) {
    return { success: false, error: e.message };
  }
}
```

---

## 3. 画像生成

### 基本パターン

```javascript
function generateImage(prompt) {
  const apiKey = getApiKey();
  if (!apiKey) {
    return { success: false, error: 'APIキーが設定されていません' };
  }

  // ※モデル名は公式ドキュメントで最新を確認
  const model = 'gemini-2.0-flash-exp-image-generation';
  const url = `https://generativelanguage.googleapis.com/v1beta/models/${model}:generateContent?key=${apiKey}`;

  const payload = {
    contents: [{
      parts: [{ text: prompt }]
    }],
    generationConfig: {
      responseModalities: ['IMAGE']
    }
  };

  try {
    const response = UrlFetchApp.fetch(url, {
      method: 'post',
      contentType: 'application/json',
      payload: JSON.stringify(payload),
      muteHttpExceptions: true
    });

    const result = JSON.parse(response.getContentText());

    if (result.error) {
      console.error('Image generation error:', result.error);
      return { success: false, error: result.error.message };
    }

    // 画像データの取得
    const parts = result.candidates?.[0]?.content?.parts;
    if (!parts) {
      return { success: false, error: '画像が生成されませんでした' };
    }

    const imagePart = parts.find(p => p.inlineData);
    if (!imagePart) {
      return { success: false, error: '画像データが見つかりません' };
    }

    return {
      success: true,
      imageBase64: imagePart.inlineData.data,
      mimeType: imagePart.inlineData.mimeType
    };
  } catch (e) {
    console.error('generateImage error:', e);
    return { success: false, error: e.message };
  }
}
```

### クライアント側での画像表示

```html
<img id="generatedImage" style="display: none; max-width: 100%;">
<a id="downloadLink" style="display: none;">ダウンロード</a>

<script>
function displayGeneratedImage(result) {
  if (!result.success) {
    showToast('画像生成に失敗しました: ' + result.error, 'danger');
    return;
  }

  const img = document.getElementById('generatedImage');
  const link = document.getElementById('downloadLink');

  const dataUrl = `data:${result.mimeType};base64,${result.imageBase64}`;

  img.src = dataUrl;
  img.style.display = 'block';

  link.href = dataUrl;
  link.download = 'generated-image.png';
  link.style.display = 'inline-block';
  link.textContent = '画像をダウンロード';
}
</script>
```

---

## 4. 2段階生成パターン

テキストAIでレイアウト設計 → 画像AIで生成の2段階パターン。

### 例: 色紙メーカー

```javascript
function generateShikishi(comments, theme) {
  const apiKey = getApiKey();
  if (!apiKey) {
    return { success: false, error: 'APIキーが設定されていません' };
  }

  // Step 1: Geminiでレイアウト設計
  const layoutPrompt = `
以下のコメントを色紙にレイアウトしてください。
テーマ: ${theme}
コメント:
${comments.map(c => `- ${c.author}: ${c.text}`).join('\n')}

レイアウトの指示を日本語で詳細に記述してください。
`;

  const layoutResult = generateText(layoutPrompt);
  if (!layoutResult.success) {
    return { success: false, error: 'レイアウト設計に失敗: ' + layoutResult.error };
  }

  // Step 2: 画像生成
  const imagePrompt = `
${theme}テーマの日本の卒業色紙を作成してください。

レイアウト指示:
${layoutResult.text}

要件:
- 日本語テキストを美しく配置
- 高解像度で印刷品質
- 装飾要素を含める
`;

  const imageResult = generateImage(imagePrompt);
  if (!imageResult.success) {
    return { success: false, error: '画像生成に失敗: ' + imageResult.error };
  }

  return {
    success: true,
    layout: layoutResult.text,
    imageBase64: imageResult.imageBase64,
    mimeType: imageResult.mimeType
  };
}
```

---

## 5. マルチモーダル（画像入力）

```javascript
function analyzeImage(imageBase64, mimeType, prompt) {
  const apiKey = getApiKey();
  if (!apiKey) {
    return { success: false, error: 'APIキーが設定されていません' };
  }

  const model = 'gemini-2.0-flash';
  const url = `https://generativelanguage.googleapis.com/v1beta/models/${model}:generateContent?key=${apiKey}`;

  const payload = {
    contents: [{
      parts: [
        {
          inlineData: {
            mimeType: mimeType,
            data: imageBase64
          }
        },
        { text: prompt }
      ]
    }]
  };

  try {
    const response = UrlFetchApp.fetch(url, {
      method: 'post',
      contentType: 'application/json',
      payload: JSON.stringify(payload),
      muteHttpExceptions: true
    });

    const result = JSON.parse(response.getContentText());

    if (result.error) {
      return { success: false, error: result.error.message };
    }

    const text = result.candidates?.[0]?.content?.parts?.[0]?.text;
    return { success: true, text: text };
  } catch (e) {
    return { success: false, error: e.message };
  }
}
```

---

## 6. エラーハンドリング

### よくあるエラーと対処

| エラー | 原因 | 対処 |
|-------|------|------|
| `API key not valid` | APIキーが無効 | Google AI Studioで新しいキーを発行 |
| `Model not found` | モデル名が間違っている | 公式ドキュメントで最新のモデル名を確認 |
| `Quota exceeded` | レート制限 | 時間をおいて再試行 |
| `Content policy violation` | コンテンツポリシー違反 | プロンプトを修正 |
| `SAFETY` | 安全性フィルタ | プロンプトを修正 |

### リトライ処理

```javascript
function fetchWithRetry(url, options, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      const response = UrlFetchApp.fetch(url, options);
      const code = response.getResponseCode();

      if (code === 200) {
        return response;
      }

      if (code === 429) {
        // レート制限: 待機して再試行
        Utilities.sleep(Math.pow(2, i) * 1000);
        continue;
      }

      // その他のエラー
      return response;
    } catch (e) {
      if (i === maxRetries - 1) throw e;
      Utilities.sleep(Math.pow(2, i) * 1000);
    }
  }
}
```

---

## 7. ローディング表示（クライアント側）

AI処理は時間がかかるため、適切なローディング表示が必要。

```html
<button id="generateBtn" onclick="generate()">生成する</button>
<div id="loadingOverlay" class="loading-overlay" style="display: none;">
  <div class="loading-spinner"></div>
  <p>AIが生成中です...<br>しばらくお待ちください</p>
</div>

<style>
.loading-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.7);
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  z-index: 9999;
  color: white;
}

.loading-spinner {
  width: 50px;
  height: 50px;
  border: 4px solid rgba(255, 255, 255, 0.3);
  border-top-color: white;
  border-radius: 50%;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}
</style>

<script>
function generate() {
  const btn = document.getElementById('generateBtn');
  const overlay = document.getElementById('loadingOverlay');

  btn.disabled = true;
  overlay.style.display = 'flex';

  google.script.run
    .withSuccessHandler(result => {
      btn.disabled = false;
      overlay.style.display = 'none';
      handleResult(result);
    })
    .withFailureHandler(e => {
      btn.disabled = false;
      overlay.style.display = 'none';
      showToast('エラー: ' + e.message, 'danger');
    })
    .generateWithAI();
}
</script>
```
