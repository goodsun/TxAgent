# GitHub Pages デプロイメント手順

## 設置先
https://goodsun.github.io/TxAgent/prototype/

## デプロイ手順

### 1. GitHubリポジトリの作成
```bash
# リポジトリ名: TxAgent
git init
git add .
git commit -m "Initial commit: MetaMask URL Bridge prototype"
git remote add origin https://github.com/goodsun/TxAgent.git
git push -u origin main
```

### 2. ディレクトリ構造
```
TxAgent/
└── prototype/
    ├── index.html           # DApp側
    ├── metamask-tx.html     # MetaMaskで開く確認ページ
    └── README.md           # 説明書
```

### 3. GitHub Pages設定
1. GitHubのリポジトリ設定を開く
2. Settings → Pages
3. Source: Deploy from a branch
4. Branch: main / root
5. Save

### 4. 動作確認URL
- DApp側: https://goodsun.github.io/TxAgent/prototype/index.html
- MetaMask確認ページ: https://goodsun.github.io/TxAgent/prototype/metamask-tx.html

## 使い方

### モバイルでの利用
1. Chrome/Safariで https://goodsun.github.io/TxAgent/prototype/ を開く
2. 「購入する」ボタンをタップ
3. URLがコピーされる
4. MetaMaskアプリを開く
5. ブラウザタブでURLを貼り付け
6. 日本語で内容確認して承認

### PCでのテスト
1. https://goodsun.github.io/TxAgent/prototype/ を開く
2. 「購入する」ボタンをクリック
3. コピーされたURLを新しいタブで開く
4. MetaMaskがインストールされていれば承認可能

## 注意事項
- HTTPS環境なのでクリップボードAPIが正常動作
- MetaMaskブラウザでの動作確認が必要
- 確認コードによる改ざん防止機能付き