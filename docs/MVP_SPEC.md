# MetaMask Clipboard Bridge - MVP仕様

## コアコンセプト
「何をしようとしているか」を完全に可視化したセキュアなトランザクション

## MVP実装仕様

### 1. 基本データ構造
```typescript
interface ClipboardTransaction {
  // 必須：トランザクション実行データ
  tx: {
    to: string;      // 送信先アドレス
    data: string;    // calldata
    value: string;   // 送金額（wei）
  };
  
  // 必須：人間が読める説明
  readable: {
    action: string;           // "NFTを転送" 
    targetContract: string;   // "CryptoKitties"
    summary: string;          // "CryptoKitties #1234 を山田さんに転送"
    details: {               // 詳細パラメータ
      [key: string]: any;
    }
  };
  
  // 必須：セキュリティ情報
  security: {
    origin: string;          // "https://example.com"
    timestamp: number;       // Unix timestamp
    chainId: number;         // 1 (mainnet), 137 (polygon)等
  };
}
```

### 2. DApp側実装例
```javascript
// NFT転送の例
const txData = {
  tx: {
    to: "0x06012c8cf97BEaD5deAe237070F9587f8E7A266d", // CryptoKitties
    data: "0xa9059cbb000000000000000000000000...",     // transfer calldata
    value: "0x0"
  },
  readable: {
    action: "NFT転送",
    targetContract: "CryptoKitties", 
    summary: "CryptoKitties #1234 を 田中さん(0xabc...def) に転送",
    details: {
      tokenId: "1234",
      tokenName: "Fluffy Cat",
      recipient: "0xabc...def",
      recipientENS: "tanaka.eth"
    }
  },
  security: {
    origin: "https://cryptokitties.co",
    timestamp: Date.now(),
    chainId: 1
  }
};

// Base64エンコードしてコピー
const encoded = btoa(JSON.stringify(txData));
await navigator.clipboard.writeText(encoded);
```

### 3. MetaMask側での表示

```
┌─────────────────────────────────────┐
│   🔐 トランザクション確認           │
├─────────────────────────────────────┤
│                                     │
│ 📋 操作: NFT転送                    │
│                                     │
│ 🎯 対象: CryptoKitties              │
│                                     │
│ 📝 内容:                            │
│ CryptoKitties #1234 を              │
│ 田中さん(tanaka.eth) に転送         │
│                                     │
│ 🔍 詳細:                            │
│ • Token: Fluffy Cat (#1234)         │
│ • 送信先: 0xabc...def               │
│ • 送信元: cryptokitties.co          │
│                                     │
│ ⚠️  ガス代: 約 0.003 ETH            │
│                                     │
│ [拒否]            [確認して署名]    │
└─────────────────────────────────────┘
```

### 4. さらなる可読性向上機能

#### A. コントラクト検証バッジ
```javascript
readable: {
  verification: {
    etherscan: true,      // ✓ Etherscan認証済み
    audited: true,        // ✓ 監査済み
    knownContract: "OpenSea"  // 既知の大手サービス
  }
}
```

#### B. リスク警告
```javascript
readable: {
  warnings: [
    "大量のNFTを一度に転送しています",
    "送信先は新規作成されたアドレスです"
  ]
}
```

#### C. トランザクション効果のプレビュー
```javascript
readable: {
  effects: {
    before: "あなたはCryptoKitties #1234を所有",
    after: "田中さんがCryptoKitties #1234を所有",
    gasEstimate: "0.003 ETH"
  }
}
```

## 実装の利点

1. **完全な透明性**: ユーザーは何が起きるか完全に理解できる
2. **詐欺防止**: 怪しいコントラクトや操作が一目瞭然
3. **使いやすさ**: 技術に詳しくないユーザーでも安心
4. **既存MetaMask互換**: 大きな改修なしで実装可能

## 次のステップ

1. **プロトタイプ開発**
   - シンプルなERC20転送から開始
   - 徐々に複雑なトランザクションに対応

2. **MetaMask Snap開発**
   - クリップボードからの読み取り
   - 可読化UIの実装

3. **セキュリティ監査**
   - データ改ざん防止
   - フィッシング対策

これなら「何をしているか分からないまま署名」という問題を完全に解決できます！