# セキュリティダイジェストによる改ざん防止

## 概要
送信先アドレス + コールデータから生成した短いダイジェスト（6文字）を使用して、データの改ざんを防止する仕組み。

## ダイジェスト生成アルゴリズム

### 1. 基本的な実装
```javascript
function generateDigest(to, data) {
  // 1. 送信先とデータを結合
  const combined = to.toLowerCase() + data.toLowerCase();
  
  // 2. SHA256でハッシュ化
  const hash = crypto.createHash('sha256').update(combined).digest('hex');
  
  // 3. Base36エンコード（0-9, A-Z）で6文字に変換
  const digest = parseInt(hash.slice(0, 10), 16)
    .toString(36)
    .toUpperCase()
    .slice(0, 6)
    .padStart(6, '0');
    
  return digest;
}

// 例: "3A8K2M"
```

### 2. より安全な実装（秘密鍵付き）
```javascript
function generateSecureDigest(to, data, userSecret) {
  // ユーザー固有のシークレットを含める
  const combined = userSecret + to.toLowerCase() + data.toLowerCase();
  
  // HMAC-SHA256でハッシュ化
  const hmac = crypto.createHmac('sha256', userSecret);
  hmac.update(combined);
  const hash = hmac.digest('hex');
  
  // 6文字のダイジェストに変換
  return hash.slice(0, 6).toUpperCase();
}
```

## 使用フロー

### DApp側
```javascript
// トランザクションデータ作成
const txData = {
  to: "0x1234...",
  data: "0xa9059cbb...",
  value: "0x0"
};

// ダイジェスト生成
const digest = generateDigest(txData.to, txData.data);

// 画面に表示
console.log(`確認コード: ${digest}`);
// 例: "確認コード: 3A8K2M"
```

### MetaMask側
```javascript
// クリップボードからデータ取得
const receivedData = JSON.parse(atob(clipboardData));

// ダイジェスト再計算
const calculatedDigest = generateDigest(
  receivedData.to, 
  receivedData.data
);

// ユーザーに確認を求める
showConfirmDialog({
  message: "確認コードを入力してください",
  expectedDigest: calculatedDigest
});
```

## UIデザイン

### DApp側の表示
```
┌─────────────────────────────────┐
│ トランザクションを送信します     │
│                                 │
│ 🔐 確認コード: 3A8K2M          │
│                                 │
│ このコードがMetaMaskに表示される │
│ コードと一致することを確認して   │
│ ください                       │
│                                 │
│ [MetaMaskで続ける]              │
└─────────────────────────────────┘
```

### MetaMask側の確認画面
```
┌─────────────────────────────────┐
│ 🔐 セキュリティ確認             │
│                                 │
│ 元の画面の確認コードと          │
│ 一致していることを確認:         │
│                                 │
│ ╔═══╦═══╦═══╦═══╦═══╦═══╗     │
│ ║ 3 ║ A ║ 8 ║ K ║ 2 ║ M ║     │
│ ╚═══╩═══╩═══╩═══╩═══╩═══╝     │
│                                 │
│ ⚠️ コードが異なる場合は         │
│ 絶対に承認しないでください       │
│                                 │
│ ❓ コードは一致していますか？    │
│                                 │
│ [いいえ（拒否）] [はい（承認）]  │
└─────────────────────────────────┘
```

## 改良案

### 1. ビジュアルダイジェスト
```javascript
// 文字だけでなく、色や絵文字も使用
function generateVisualDigest(to, data) {
  const hash = generateDigest(to, data);
  const emojis = ['🟦', '🟩', '🟨', '🟧', '🟥', '🟪'];
  const colors = ['blue', 'green', 'yellow', 'orange', 'red', 'purple'];
  
  return {
    code: hash,
    visual: hash.split('').map((char, i) => ({
      char: char,
      emoji: emojis[char.charCodeAt(0) % 6],
      color: colors[i]
    }))
  };
}
```

### 2. 時限ダイジェスト
```javascript
// 時間制限付きダイジェスト
function generateTimedDigest(to, data) {
  const timestamp = Math.floor(Date.now() / 60000); // 1分単位
  const combined = timestamp + to + data;
  return generateDigest(combined);
}
```

## メリット
1. **簡単**: 6文字を見比べるだけで確認完了
2. **安全**: データが1文字でも変わればダイジェストが変化
3. **視覚的**: 人間が確認しやすい
4. **汎用的**: どんなトランザクションにも適用可能
5. **記憶不要**: 覚える必要なし、見比べるだけ

## デメリット
1. **画面切替**: DApp画面とMetaMask画面を見比べる必要がある
2. **衝突**: 理論的には異なるデータで同じダイジェストになる可能性

## さらなる改善案

### スクリーンショット方式
```javascript
// DApp側で確認コードを含む画面をスクリーンショット可能にする
function showDigestWithScreenshot(digest) {
  return `
    <div id="security-digest-card" style="
      background: linear-gradient(45deg, #f0f0f0, #ffffff);
      border: 2px solid #4CAF50;
      padding: 20px;
      border-radius: 10px;
      text-align: center;
    ">
      <h3>🔐 セキュリティ確認コード</h3>
      <div style="
        font-size: 32px;
        font-weight: bold;
        letter-spacing: 8px;
        color: #2196F3;
        background: #f5f5f5;
        padding: 10px;
        border-radius: 5px;
        margin: 10px 0;
      ">
        ${digest}
      </div>
      <p style="font-size: 12px; color: #666;">
        このコードをスクリーンショットして<br>
        MetaMaskで確認してください
      </p>
    </div>
  `;
}
```

### 画面分割表示（可能な端末の場合）
- Split View対応端末では両画面を同時表示
- Picture-in-Picture機能で確認コードを常時表示

## 実装の簡易性

### 最小限の実装
```javascript
// DApp側: たった数行で実装可能
const digest = generateDigest(to, data);
alert(`確認コード: ${digest}\n\nこのコードをMetaMaskで確認してください`);

// MetaMask側: シンプルな確認ダイアログ
const expectedDigest = generateDigest(tx.to, tx.data);
if (confirm(`確認コード: ${expectedDigest}\n\n元の画面と同じコードですか？`)) {
  // 承認処理
} else {
  // 拒否処理
}
```

この最小実装でも十分なセキュリティを提供できます。