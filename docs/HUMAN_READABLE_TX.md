# トランザクション内容の可読化方法

## 1. メタデータ付与方式（推奨）
```json
{
  "to": "0x1234...",
  "data": "0xa9059cbb...",
  "value": "0x0",
  "metadata": {
    "action": "NFT転送",
    "contract": "CryptoKitties",
    "method": "transfer",
    "params": {
      "recipient": "0xabcd...",
      "tokenId": "12345"
    },
    "description": "CryptoKitties #12345 を 0xabcd... に転送"
  }
}
```

## 2. ABI付与方式
```json
{
  "to": "0x1234...",
  "data": "0xa9059cbb...",
  "value": "0x0",
  "abi": {
    "name": "transfer",
    "inputs": [
      {"name": "to", "type": "address", "value": "0xabcd..."},
      {"name": "tokenId", "type": "uint256", "value": "12345"}
    ]
  }
}
```

## 3. 4byte Directory統合
```json
{
  "to": "0x1234...",
  "data": "0xa9059cbb...",
  "value": "0x0",
  "functionSignature": "transfer(address,uint256)",
  "decodedParams": {
    "to": "0xabcd...",
    "tokenId": "12345"
  }
}
```

## 4. コントラクト情報付与
```json
{
  "to": "0x1234...",
  "data": "0xa9059cbb...",
  "value": "0x0",
  "contractInfo": {
    "name": "Uniswap V3 Router",
    "verified": true,
    "type": "DEX"
  },
  "txInfo": {
    "action": "スワップ",
    "from": "ETH",
    "to": "USDC",
    "amount": "1.5 ETH"
  }
}
```

## 実装例：完全な可読化データ
```javascript
async function createReadableTransaction(to, data, value) {
  // 関数セレクタを抽出（最初の4バイト）
  const selector = data.slice(0, 10);
  
  // 既知のコントラクトか確認
  const contractInfo = await getContractInfo(to);
  
  // ABIから関数情報を取得
  const functionInfo = contractInfo.abi.find(
    f => f.signature.startsWith(selector)
  );
  
  // パラメータをデコード
  const decodedParams = web3.eth.abi.decodeParameters(
    functionInfo.inputs,
    data.slice(10)
  );
  
  return {
    // 基本的なトランザクションデータ
    to,
    data,
    value,
    
    // 人間が読める情報
    readable: {
      contract: {
        name: contractInfo.name,
        type: contractInfo.type,
        verified: contractInfo.verified
      },
      function: {
        name: functionInfo.name,
        signature: functionInfo.signature
      },
      parameters: functionInfo.inputs.map((input, i) => ({
        name: input.name,
        type: input.type,
        value: decodedParams[i],
        humanValue: formatValue(decodedParams[i], input.type)
      })),
      summary: generateSummary(contractInfo, functionInfo, decodedParams)
    }
  };
}

// 要約文の生成
function generateSummary(contract, func, params) {
  // コントラクトタイプと関数名に基づいて要約を生成
  if (contract.type === 'NFT' && func.name === 'transfer') {
    return `${contract.name} #${params.tokenId}を${params.to}に転送`;
  }
  if (contract.type === 'DEX' && func.name === 'swap') {
    return `${params.amountIn} ${params.tokenIn}を${params.tokenOut}にスワップ`;
  }
  // ... 他のパターン
}
```

## メタマスク側での表示イメージ
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🔐 トランザクション確認

📄 操作内容：NFT転送
🏷️ コントラクト：CryptoKitties (検証済み✓)
🎯 関数：transfer(address,uint256)

📊 詳細：
  • 送信先: 0xabcd...ef01
  • Token ID: #12345
  
💬 要約：
CryptoKitties #12345 を 0xabcd...ef01 に転送します

⚠️ ガス代見積もり：0.003 ETH
━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

## セキュリティ考慮事項
1. **検証済みコントラクト表示**: Etherscan等で検証済みかを表示
2. **既知の詐欺コントラクト警告**: ブラックリストとの照合
3. **異常なパラメータの警告**: 通常と異なる値の検出
4. **シミュレーション結果**: 実行結果の事前表示