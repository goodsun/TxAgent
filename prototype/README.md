# MetaMask Clipboard Bridge プロトタイプ

## 概要
HTML+JavaScriptのみで実装したシンプルなプロトタイプです。

## ファイル構成
- `index.html` - DApp側（NFTマーケットプレイス）
- `metamask-receiver.html` - MetaMask側（トランザクション確認画面）

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
1. ブラウザで `http://localhost:8000/index.html` を開く
2. 「購入する」ボタンをクリック
3. 確認コード（例: 3A8K2M）が表示される
4. 「MetaMaskへ進む」ボタンをクリック（クリップボードにデータがコピーされる）
5. 別タブで `http://localhost:8000/metamask-receiver.html` を開く
6. 自動的にクリップボードからデータが読み込まれ、確認コードが表示される
7. 確認コードが一致していることを確認して「承認」をクリック

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
- クリップボードAPIを使用

## 注意事項
- HTTPS環境またはlocalhostでのみクリップボードAPIが動作します
- 実際のMetaMaskとの連携にはMetaMask Snapの開発が必要です
- ダイジェスト生成は簡易版です（本番環境ではSHA256を使用推奨）

## 今後の改善点
1. MetaMask Snapとしての実装
2. より高度なトランザクションタイプへの対応
3. エラーハンドリングの強化
4. UIデザインの洗練