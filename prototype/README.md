# MetaMask Clipboard Bridge プロトタイプ

## 概要
URL直接コピー方式で実装したシンプルなプロトタイプです。

## ファイル構成
- `index.html` - DApp側（NFTマーケットプレイス）
- `metamask-tx.html` - MetaMaskブラウザで開くトランザクション確認ページ
- `metamask-receiver.html` - （旧バージョン）

## 動作確認方法

### 1. ローカルサーバーの起動
```bash
# Python 3の場合
python -m http.server 8000

# Python 2の場合
python -m SimpleHTTPServer 8000

# Node.jsの場合
npx http-server -p 8000
```

### 2. 動作確認手順

#### 実際の使い方（モバイル想定）
1. Chrome/Safariで `http://localhost:8000/index.html` を開く
2. 「購入する」ボタンをクリック
3. URLが自動的にコピーされ、確認コード（例: 3A8K2M）が表示される
4. MetaMaskアプリを開く
5. ブラウザタブを開き、アドレスバーにURLを貼り付け
6. 日本語でトランザクション内容が表示される
7. 確認コードが一致していることを確認して「承認」をクリック

#### デモ用（PC）
1. ブラウザで `http://localhost:8000/index.html` を開く
2. 「購入する」ボタンをクリック
3. 表示されたURLをコピー
4. 新しいタブでURLを開く（MetaMaskがインストールされている場合は承認可能）

## 実装のポイント

### セキュリティ
- 送信先アドレス + calldataからダイジェストを生成
- 6文字の確認コードで改ざんを検知

### 可読性
- トランザクション内容を日本語で表示
- NFT名、価格、操作内容を明確に表示

### シンプルさ
- 外部ライブラリ不要
- 純粋なHTML+JavaScriptのみ
- URLをコピーするだけ

### 新しいフロー
1. **通常ブラウザ**: トランザクションデータを含むURLを生成
2. **クリップボード**: URLをコピー（データではなくURL）
3. **MetaMaskブラウザ**: URLを開いて日本語で確認

## 注意事項
- HTTPS環境またはlocalhostでのみクリップボードAPIが動作します
- 実際のMetaMaskとの連携にはMetaMask Snapの開発が必要です
- ダイジェスト生成は簡易版です（本番環境ではSHA256を使用推奨）

## 今後の改善点
1. MetaMask Snapとしての実装
2. より高度なトランザクションタイプへの対応
3. エラーハンドリングの強化
4. UIデザインの洗練