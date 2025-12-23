# Facebook友達限定オークションサイト計画

このリポジトリでは、Facebookアカウントによる認証を用いて「友達限定」で参加できるオークションサイトの開発計画をまとめています。以下は、初期段階の要件とアーキテクチャ方針です。

## 目的
- Facebookの友達（もしくは特定の友達リスト）に限定したクローズドなマーケットプレイスを構築する。
- シンプルなオークション体験（出品・入札・落札）を提供する。
- 信頼できるユーザー基盤（実名・既知の友人）を前提とし、コミュニティ内で安心して取引できる環境を整える。

## MVP機能
1. **Facebookログイン**
   - Facebook Login (OAuth 2.0) を利用し、ユーザー登録とログインを統合する。
   - 初回ログイン時にFacebook Graph APIから友達リストを取得し、許可されている友達のみサイト利用を許可する。
2. **ユーザー管理**
   - Facebook IDとアプリ内ユーザーIDのマッピング。
   - プロフィール情報（表示名、アイコンURL）を保存。
3. **出品機能**
   - 友達限定で出品できるアイテム（タイトル、説明、画像、開始価格、即決価格など）。
   - 入札期限の設定。
4. **入札機能**
   - 認証済みの友達のみ入札可能。
   - 自動延長（例：終了1分前の入札で5分延長）などのオプション。
5. **落札・取引管理**
   - 落札通知（アプリ内通知 + Facebook Messenger通知など）。
   - 取引ステータス管理（支払い待ち、発送待ち、完了）。
6. **安全性・信頼性**
   - 通報機能やブラックリスト機能。
   - 監査ログ保存。

## システム構成案
```
ユーザー -> (Next.js/React SPA) -> API Gateway (Express/NestJS) -> サービス層
                                               |
                                               +--> Facebook Graph API (OAuth, Friends API)
                                               +--> DB (PostgreSQL)
                                               +--> ストレージ (S3等)
```

### フロントエンド
- Next.js + TypeScript + Tailwind CSS。
- Facebook Login JavaScript SDKを利用しOAuthフローを開始。
- GraphQLまたはRESTでバックエンドと通信。

### バックエンド
- Node.js (NestJSまたはExpress) + TypeScript。
- Passport.js + passport-facebook-tokenでFacebookアクセストークンを検証。
- Prisma ORMでDBアクセスを行う。

### データベース設計（例）
- `users`
  - `id` (UUID)
  - `facebook_id`
  - `display_name`
  - `avatar_url`
  - `friend_whitelist` (JSONB: 利用を許可した友達ID)
- `listings`
  - `id`
  - `seller_id`
  - `title`
  - `description`
  - `start_price`
  - `buy_now_price`
  - `current_price`
  - `status` (draft/active/closed)
  - `end_at`
- `bids`
  - `id`
  - `listing_id`
  - `bidder_id`
  - `amount`
  - `created_at`
- `transactions`
  - `id`
  - `listing_id`
  - `buyer_id`
  - `seller_id`
  - `status`

### 友達限定ロジック
1. Facebook OAuthで取得したアクセストークンを用いて `/me/friends` エンドポイントを呼び出し、アプリ利用許可済みの友達一覧を取得。
2. アプリ側のDBに保存されているユーザーIDと照合し、友達同士である場合のみ入札・閲覧を許可。
3. `friend_whitelist` カラムに「自分が許可した友達ID」を保持し、相互許可であることを入札条件にする。

### 開発ステップ
1. Facebook for Developersでアプリ登録。
2. OAuthリダイレクトURL設定、App Reviewで`user_friends`権限申請。
3. バックエンドでFacebookアクセストークン検証ロジック実装。
4. 初回ログイン時にユーザーをDBへ作成・更新。
5. 友達関係の同期処理を定期的に実行（Webhookまたはバッチ）。
6. 出品・入札APIを設計・実装。
7. UI作成：出品一覧、詳細、入札フォーム、取引管理画面。
8. 通知機能（メール、Messenger、Push）を追加。

## セキュリティとコンプライアンス
- Facebookのプラットフォームポリシーに従う。
- OAuthトークンは短命（short-lived token）で取得し、必要に応じてリフレッシュ。
- HTTPS必須、JWT署名に鍵管理。
- 利用規約・プライバシーポリシー整備。

## 今後の拡張アイデア
- 友達グループごとに公開範囲を切り替えられるリスト機能。
- 取引完了後のレビュー・評価システム。
- Facebook以外のSNSログイン拡張（Instagram、WhatsApp等）。

この計画をベースに具体的な実装を進めていきます。
