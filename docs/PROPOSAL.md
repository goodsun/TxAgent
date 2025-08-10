# MetaMask Clipboard Bridge - プロポーザル

## 概要
スマートフォンにおいて、通常のブラウザでDAppsを快適に閲覧しながら、トランザクション発行時のみMetaMaskと連携する新しい仕組みを提案する。

## 背景と課題
- **現状**: MetaMaskブラウザ内でのDApps利用は制限が多く、UXが劣る
- **ニーズ**: ネイティブブラウザの快適性を保ちながら、Web3機能を利用したい
- **制約**: モバイル環境では、異なるアプリ間の直接通信は困難
- **最大の問題**: 現在のMetaMaskでは「0xa9059cbb...」のような理解不能な16進数データに署名させられ、ユーザーは何をしているか分からない

## 提案手法: クリップボードブリッジ方式

### 基本コンセプト
1. **閲覧**: 通常のブラウザ（Chrome/Safari等）でDAppsを閲覧
2. **認証**: MetaMask SDKでウォレットアドレスのみ取得
3. **署名**: トランザクションデータをクリップボード経由でMetaMaskに渡す
4. **可読化**: すべてのトランザクションを日本語で完全に説明

### 技術アーキテクチャ

```
[ネイティブブラウザ]
    ↓
[DApps + MetaMask SDK]
    ↓
[トランザクション生成]
    ↓
[クリップボードへコピー]
    ↓
[MetaMaskへ遷移]
    ↓
[ペースト&署名]
```

### データフォーマット案

```json
{
  "version": "1.0",
  "type": "eth_sendTransaction",
  "tx": {
    "to": "0x...",
    "value": "0x...",
    "data": "0x..."
  },
  "readable": {
    "action": "NFT転送",
    "targetContract": "CryptoKitties",
    "summary": "CryptoKitties #1234 を 田中さん(tanaka.eth) に転送",
    "details": {
      "tokenId": "1234",
      "tokenName": "Fluffy Cat",
      "recipient": "0xabc...def",
      "recipientENS": "tanaka.eth"
    },
    "warnings": [],
    "verification": {
      "etherscan": true,
      "audited": true
    }
  },
  "security": {
    "origin": "https://cryptokitties.co",
    "timestamp": 1234567890,
    "chainId": 1
  }
}
```

### 実装フロー

#### Phase 1: 初回接続
1. DAppsでMetaMask SDKを初期化
2. `eth_requestAccounts`でアドレス取得のみ実行
3. セッション情報を保存

#### Phase 2: トランザクション発行
1. トランザクションパラメータを構築
2. 専用フォーマットでエンコード（Base64）
3. クリップボードにコピー
4. カスタムスキーム or Universal Linkでメタマスクを起動
5. ユーザーがメタマスク内でペースト&確認&署名

#### Phase 3: 結果確認
1. トランザクションハッシュをコピー
2. 元のブラウザに戻る
3. ハッシュをペーストして結果確認

## メリット
- **高速**: アプリ切り替えが最小限
- **安全**: トランザクションデータの改ざんが困難
- **シンプル**: 複雑な通信プロトコル不要
- **互換性**: 既存のMetaMaskアプリで実現可能
- **完全な可読性**: トランザクション内容を日本語で完全に理解できる

## デメリット・課題
- **UX**: コピー&ペーストの手間
- **制限**: 大量のデータには不向き
- **検証**: MetaMask側でのデータ検証機能が必要

## セキュリティ考慮事項
1. **署名検証**: DApps側で生成したデータの完全性確保
2. **タイムスタンプ**: リプレイ攻撃防止
3. **ドメイン検証**: フィッシング対策
4. **可読性による詐欺防止**: 怪しい操作が一目瞭然

## 革新的な特徴: 完全な可読化

### 現在のMetaMaskの問題
```
トランザクションデータ:
0xa9059cbb0000000000000000000000001234567890abcdef...
```
→ **ユーザーには全く理解できない**

### 本提案での表示
```
📋 操作: NFT転送
🎯 対象: CryptoKitties
📝 内容: CryptoKitties #1234 を 田中さん(tanaka.eth) に転送
⚠️ ガス代: 約 0.003 ETH
```
→ **誰でも完全に理解できる**

この可読化により：
- 詐欺被害を事前に防止
- 初心者でも安心して利用可能
- トランザクションの透明性を確保

## 実装ロードマップ
1. **Phase 1**: プロトタイプ実装（1週間）
   - 基本的なコピー&ペースト機能
   - シンプルな送金トランザクション

2. **Phase 2**: セキュリティ強化（2週間）
   - データ検証機能
   - エラーハンドリング

3. **Phase 3**: UX改善（2週間）
   - ワンタップコピー
   - 状態管理の最適化

## 次のステップ
1. 技術検証のためのPoCを実装
2. MetaMaskチームへのフィードバック収集
3. コミュニティでの意見募集

## 参考実装イメージ

```javascript
// 1. 基本的な送金トランザクション
async function sendTransaction(params) {
  const txData = {
    version: "1.0",
    type: "eth_sendTransaction",
    params: params,
    metadata: {
      dappName: "MyDApp",
      dappUrl: window.location.origin,
      timestamp: Date.now()
    }
  };
  
  const encoded = btoa(JSON.stringify(txData));
  await navigator.clipboard.writeText(encoded);
  
  // MetaMaskを起動
  window.location.href = 'metamask://transaction';
}

// 2. NFT転送の実装例
async function transferNFT(contractAddress, tokenId, toAddress) {
  // ERC721のtransferFromメソッドを呼び出す
  const transferData = web3.eth.abi.encodeFunctionCall({
    name: 'transferFrom',
    type: 'function',
    inputs: [{
      type: 'address',
      name: 'from'
    }, {
      type: 'address',
      name: 'to'
    }, {
      type: 'uint256',
      name: 'tokenId'
    }]
  }, [userAddress, toAddress, tokenId]);

  const txData = {
    version: "1.0",
    type: "eth_sendTransaction",
    params: {
      from: userAddress,
      to: contractAddress,
      value: "0x0",
      data: transferData,
      gas: "0x15f90" // 約90,000 gas
    },
    metadata: {
      dappName: "NFT Transfer App",
      dappUrl: window.location.origin,
      timestamp: Date.now(),
      description: `Transfer NFT #${tokenId} to ${toAddress}`
    }
  };
  
  const encoded = btoa(JSON.stringify(txData));
  await navigator.clipboard.writeText(encoded);
  window.location.href = 'metamask://transaction';
}

// 3. NFTミントの実装例
async function mintNFT(contractAddress, mintPrice) {
  // NFTコントラクトのmintメソッドを呼び出す
  const mintData = web3.eth.abi.encodeFunctionCall({
    name: 'mint',
    type: 'function',
    inputs: [{
      type: 'uint256',
      name: 'quantity'
    }]
  }, [1]); // 1個ミント

  const txData = {
    version: "1.0",
    type: "eth_sendTransaction",
    params: {
      from: userAddress,
      to: contractAddress,
      value: web3.utils.toHex(mintPrice), // ミント価格（Wei）
      data: mintData,
      gas: "0x30d40" // 約200,000 gas（ミントは高め）
    },
    metadata: {
      dappName: "NFT Minting DApp",
      dappUrl: window.location.origin,
      timestamp: Date.now(),
      description: "Mint 1 NFT",
      nftInfo: {
        contractName: "Cool NFT Collection",
        quantity: 1,
        priceETH: web3.utils.fromWei(mintPrice, 'ether')
      }
    }
  };
  
  const encoded = btoa(JSON.stringify(txData));
  await navigator.clipboard.writeText(encoded);
  window.location.href = 'metamask://transaction';
}

// 4. より複雑なNFTミント（ホワイトリスト付き）
async function mintWithWhitelist(contractAddress, proof, mintPrice) {
  const mintData = web3.eth.abi.encodeFunctionCall({
    name: 'whitelistMint',
    type: 'function',
    inputs: [{
      type: 'bytes32[]',
      name: 'proof'
    }, {
      type: 'uint256',
      name: 'quantity'
    }]
  }, [proof, 1]);

  const txData = {
    version: "1.0",
    type: "eth_sendTransaction",
    params: {
      from: userAddress,
      to: contractAddress,
      value: web3.utils.toHex(mintPrice),
      data: mintData,
      gas: "0x493e0" // 約300,000 gas（Merkle proof検証含む）
    },
    metadata: {
      dappName: "Exclusive NFT Drop",
      dappUrl: window.location.origin,
      timestamp: Date.now(),
      description: "Whitelist mint",
      nftInfo: {
        type: "whitelist",
        contractName: "Premium Collection"
      }
    }
  };
  
  const encoded = btoa(JSON.stringify(txData));
  await navigator.clipboard.writeText(encoded);
  window.location.href = 'metamask://transaction';
}
```