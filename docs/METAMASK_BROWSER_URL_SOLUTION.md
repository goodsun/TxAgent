# MetaMaskブラウザで専用URL方式

## 解決策：超シンプル

### 1. DApp側の実装
```javascript
// トランザクション承認用の専用URLを生成
async function createMetaMaskURL() {
  // トランザクションデータを一時保存
  const txId = generateUUID();
  const txData = {
    to: '0x06012c8cf97BEaD5deAe237070F9587f8E7A266d',
    value: '0x16345785d8a0000',
    data: '0xa9059cbb...'
  };
  
  // サーバーに保存（または暗号化してURLに含める）
  await saveTxData(txId, txData);
  
  // MetaMaskブラウザ専用のURL
  const metamaskURL = `https://your-dapp.com/metamask-tx/${txId}`;
  
  // 確認コード生成
  const digest = generateDigest(txData);
  
  return { metamaskURL, digest };
}

// 使用例
const { metamaskURL, digest } = await createMetaMaskURL();

// ユーザーに表示
alert(`
📋 MetaMaskで開いてください：
${metamaskURL}

🔐 確認コード: ${digest}
`);
```

### 2. MetaMaskブラウザ専用ページ
```html
<!-- https://your-dapp.com/metamask-tx/[txId] -->
<!DOCTYPE html>
<html>
<head>
  <title>トランザクション確認</title>
</head>
<body>
  <h1>🔐 トランザクション確認</h1>
  
  <div class="readable-info">
    <h2>📋 操作内容</h2>
    <p>NFT購入: CryptoKitties #1234</p>
    <p>価格: 0.1 ETH</p>
    <p>確認コード: <strong>3A8K2M</strong></p>
  </div>
  
  <button onclick="executeTx()">承認する</button>
  
  <script>
    // MetaMaskブラウザかチェック
    if (!window.ethereum || !window.ethereum.isMetaMask) {
      alert('このページはMetaMaskブラウザで開いてください');
      return;
    }
    
    async function executeTx() {
      // サーバーからトランザクションデータ取得
      const txData = await fetchTxData();
      
      // MetaMaskで実行（既に接続済み）
      const txHash = await ethereum.request({
        method: 'eth_sendTransaction',
        params: [txData]
      });
      
      alert('トランザクション送信完了: ' + txHash);
    }
  </script>
</body>
</html>
```

## フロー

1. **通常ブラウザ（Chrome/Safari）**
   - DAppで「購入」ボタンクリック
   - 専用URLと確認コードが表示される
   - URLをコピー

2. **MetaMaskブラウザ**
   - URLを貼り付けて開く
   - 日本語で内容確認
   - 確認コードを照合
   - 「承認」ボタンで実行

## メリット
- ✅ **追加インストール不要**
- ✅ **MetaMask側の改修不要**
- ✅ **完全な可読化が可能**
- ✅ **今すぐ実装可能**

## さらにシンプルな実装

```javascript
// URLに全データを含める（URLエンコード）
function createDirectURL(tx, readable) {
  const data = {
    tx: tx,
    readable: readable,
    digest: generateDigest(tx)
  };
  
  const encoded = btoa(JSON.stringify(data));
  return `https://your-dapp.com/metamask?data=${encoded}`;
}
```

## 実装例：ワンクリックコピー
```html
<div class="metamask-instruction">
  <p>MetaMaskブラウザで開いてください：</p>
  <div class="url-box">
    <input id="metamask-url" value="${metamaskURL}" readonly>
    <button onclick="copyURL()">URLをコピー</button>
  </div>
  <p>確認コード: <strong>${digest}</strong></p>
</div>
```

これなら：
1. DApp開発者が自由に実装可能
2. MetaMaskの協力不要
3. 完全な日本語表示
4. セキュリティも確保

シンプルで現実的な解決策です！