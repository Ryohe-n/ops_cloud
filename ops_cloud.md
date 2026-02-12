# KevLumos クラウド公開版  
運用者向けマニュアル（repo版・正本）

---

## 1. 本書について

### 1.1 本書の目的
本書は、KevLumos クラウド公開版を  
**運用担当者が安全かつ継続的に運用するために必要な判断観点と一次対応指針**  
を提供することを目的とする。

本書は **運用フェーズ専用マニュアル** であり、  
実装手順書・再構築手順書ではない。

---

### 1.2 本書の対象読者
- KevLumos クラウド公開版の運用を担当する者
- AWS 環境および onprem 環境への操作権限を持つ運用担当者

---

### 1.3 本書のスコープ
本書は以下を扱う。

- 日常運用における状態確認
- 障害・異常時の一次切り分け観点
- Cognito 利用者ユーザー管理に関する運用判断範囲
- secret / 機密情報の **参照・判断ルール**

以下は本書の対象外とする。

- 再構築・再デプロイ手順（D-01）
- upstream 更新追従や構成変更判断
- IAM / ロール設計・権限設計
- 運用オンボーディング設計
- Runbook / チェックリスト設計
- secret 運用ルールそのものの設計

---

### 1.4 記載方針（運用ドキュメントとして）
本書では、

- **何を確認すべきか**
- **どこまでが運用担当者の判断範囲か**
- **どの資料を参照すべきか**

を明確にする。

具体的な画面操作手順やコマンドは Appendix に集約し、  
本文には **判断軸と切り分け順序のみ** を記載する。

---

### 1.5 repo版と配布版
本書は **repo版（正本）** として Git リポジトリで管理される。

repo版の特徴:
- secret / 機密情報の **実値を含まない**
- Secret Management Register を参照する前提で成立する
- 本書が唯一の正本である

運用現場で使用する **配布版** は、  
repo版を元に secret 実値を反映して生成される派生物であり、  
正本ではない。

---

### 1.6 他ドキュメントとの関係
- 利用者向け操作説明: **B-08**
- 再構築・再配置手順: **D-01**
- secret 原本: **KevLumos Cloud Deployment ? Secret Management Register（非公開）**

---

## 2. 運用視点での全体構成

### 2.1 全体像
KevLumos クラウド公開版は、以下の層で構成される。

- Cloud frontend（CloudFront + S3）
- 認証基盤（Amazon Cognito）
- API 中継（Lambda Function URL / kcs-edge）
- onprem backend（VPN 経由）

運用担当者は、  
**どの層で問題が発生しているかを切り分けられれば十分** であり、  
内部実装の詳細理解は必須ではない。

---

### 2.2 運用担当者が触れる可能性のある範囲
- Cognito（利用者ユーザー管理）
- CloudWatch（Lambda 実行状況）
- CloudFront / S3（配信・状態確認）
- S3 / SQS（アップロード処理滞留確認）
- onprem backend（ログ・ディスク使用量）

設定変更や構成変更は本書の対象外とする。

---

## 3. 利用者ユーザー管理（Cognito）

### 3.1 運用判断の範囲
運用担当者が実施してよい操作は以下に限定される。

- 利用者ユーザーの追加
- 利用者ユーザーの無効化
- 利用者ユーザー状態の確認

以下は実施しない。

- ユーザー削除
- 認証方式の変更
- App client 設定の変更

---

### 3.2 認証トラブル時の基本確認
- ユーザーが Enabled 状態か
- User status が CONFIRMED か
- Client ID / redirect URI の不整合がないか

具体操作は **Appendix A.1** を参照する。

---

## 4. 監視・状態確認（一次判断）

### 4.1 Lambda（kcs-edge）
確認観点:
- Invocations の急増・急減
- Errors の継続発生
- Duration の異常な増加

詳細は **Appendix A.3** を参照。

---

### 4.2 CloudFront / frontend 配信
確認観点:
- Requests が継続して発生しているか
- 5xxErrorRate が高止まりしていないか

詳細は **Appendix A.2** を参照。

---

### 4.3 アップロード処理（S3 / SQS）
確認観点:
- 中間バケットに古いファイルが滞留していないか
- SQS メッセージが継続的に残っていないか

詳細は **Appendix A.4** を参照。

---

### 4.4 onprem backend
確認観点:
- 同一エラーの連続発生
- ディスク使用量の急増
- 古い一時ファイルの残留

詳細は **Appendix A.5** を参照。

---

## 5. トラブルシューティング（一次切り分け）

### 5.1 基本方針
- frontend / cloud / onprem のどこかを特定する
- 一時的か継続的かを判断する
- 判断に迷う場合は Issue に切り出す

---

### 5.2 認証・認可系
- Cognito ユーザー状態確認
- Client 設定不整合の有無

---

### 5.3 cloud / onprem 間通信
- Lambda 送信ログと backend 受信ログの対応
- internal header による拒否の有無

---

## 6. Secret / 機密情報の取り扱い（repo版専用）

### 6.1 原則
secret / 機密情報の **実値の原本** は、  
以下の非公開資料に集約される。

- **KevLumos Cloud Deployment ? Secret Management Register（非公開）**

本書および Git リポジトリには、  
secret の **論理識別子のみ** を記載する。

---

### 6.2 参照ルール
docs や Issue では、以下のように記載する。

- 「KCS_EDGE_BUCKET_NAME（Secret Management Register §2.3 参照）」
- 「ONPREM_INTERNAL_SHARED_SECRET（Secret Management Register §2.4 参照）」

実値は一切記載しない。

---

### 6.3 責務分離
以下は **本書の責務外** とする。

- secret 運用ルール設計
- ローテーション手順
- 管理体制・権限設計

これらは **B-09-D04** にて別途定義される。

---

## Appendix A. 運用時の具体操作メモ

本 Appendix は、KevLumos クラウド公開版の運用において、
運用担当者が **実際に画面やログを確認し、一次判断を行うための具体操作**
を整理したものである。

本文（第3章?第5章）に記載された判断観点に従い、
該当する Appendix 節を参照して操作を行う。

---

### A.1 認証基盤の操作
（Amazon Cognito）

#### 対象コンポーネント
- Amazon Cognito User Pool

#### 使用するコンソール
- AWS マネジメントコンソール

#### 操作の入り口（利用者ユーザーの追加）
1. AWS マネジメントコンソールにログインする
2. Amazon Cognito を開く
3. User Pools を選択する
4. 対象の User Pool を選択する
5. 「Users and groups」を開く
6. 「Create user」を選択する
7. 必要な属性を入力し、ユーザーを作成する

確認事項：
- 作成後、ユーザーが Enabled 状態であること
- User status が想定どおりであること

---

#### 操作の入り口（利用者ユーザーの無効化）
1. Amazon Cognito の User Pool を開く
2. 「Users and groups」を開く
3. 対象ユーザーを選択する
4. 「Disable」を実行する

判断の目安：
- 利用停止は削除ではなく無効化で行う
- 無効化後、Enabled = No であることを確認する

---

#### 操作の入り口（ユーザー状態確認）
1. AWS マネジメントコンソールにログインする
2. Amazon Cognito を開く
3. User Pools を選択する
4. 対象の User Pool を選択する
5. 「Users and groups」を開く
6. 対象ユーザーを検索する

#### 確認する項目（ユーザー単位）
- Enabled / Disabled
- User status
    - CONFIRMED
    - FORCE_CHANGE_PASSWORD
    - UNCONFIRMED

#### 判断の目安（運用者判断用）
- Enabled = Yes かつ User status = CONFIRMED
    → 正常に利用可能
- Enabled = No
    → 利用停止状態
- FORCE_CHANGE_PASSWORD / UNCONFIRMED
    → 初回ログイン・確認未完了の可能性

---

#### 操作の入り口（クライアント設定確認）
1. Cognito User Pool 画面で「App integration」を開く
2. App client 一覧を確認する
3. 対象 App client を選択する

#### 確認する項目（クライアント）
- Client ID
- Callback URLs（redirect URI）
- Logout URLs

#### 判断の目安
- frontend が使用している Client ID と一致している
- redirect URI が利用中の frontend URL と一致している

不一致がある場合、認証後遷移や API 利用に失敗する可能性が高い。

---

#### 運用上の注意事項（ユーザー管理）
- 利用者ユーザーの削除は行わない
- App client や認証方式の変更は行わない
- 想定外の状態が確認された場合は、運用判断として作業を止め、Issue に切り出す

---

### A.2 Cloud frontend 配信確認
（CloudFront / S3）

#### 対象コンポーネント
- CloudFront
- frontend 配信用 S3 バケット

#### 使用するコンソール / ツール
- AWS マネジメントコンソール
- Web ブラウザ

#### 操作の入り口（CloudFront）
1. AWS マネジメントコンソールにログインする
2. CloudFront を開く
3. Distributions 一覧から対象 Distribution を選択する
4. 「Monitoring」タブを開く

#### 確認するメトリクス
- Requests
- 5xxErrorRate

#### 確認する時間範囲
- 直近 15 分
    必要に応じて 1 時間まで拡張する

#### 判断の目安
- Requests が継続的に 0
    → 配信が行われていない可能性
- 5xxErrorRate が数 % 以上で継続
    → 配信異常の可能性

---

#### 補助確認（ブラウザ）
1. ブラウザで frontend に直接アクセスする
2. DevTools を開く
3. Network で HTML / JavaScript / CSS の取得を確認する
4. Console にエラーが出ていないか確認する

判断の目安：
- 5xx が継続する場合は cloud 側の問題を疑う
- リソース取得が広範に失敗する場合は配信経路問題を疑う

---

### A.3 API 中継・実行状況確認
（Lambda / CloudWatch）

#### 対象コンポーネント
- AWS Lambda（kcs-edge）
- CloudWatch Logs / Metrics

#### 使用するコンソール
- AWS マネジメントコンソール

#### 操作の入り口（メトリクス）
1. AWS Lambda を開く
2. 関数 `kcs-edge` を選択する
3. 「Monitor」タブを開く

#### 確認するメトリクス
- Invocations
- Errors
- Duration

#### 確認する時間範囲
- 直近 15 分
    必要に応じて 1 時間まで拡張する

#### 判断の目安
- Invocations が極端に少ない / 急増している
    → 呼び出し経路異常の可能性
- Errors が連続して発生
    → API 中継処理異常の可能性
- Duration が通常時より明らかに長い状態が継続
    → backend 通信遅延等の可能性

---

#### 操作の入り口（ログ）
1. Lambda 関数画面で「Logs」を開く
2. CloudWatch Logs の該当 Log stream を開く
3. エラーが単発か連続かを確認する

判断の目安：
- 同一エラーが 15 分以上継続している場合は継続障害の可能性が高い

---

### A.4 アップロード処理関連の確認
（S3 / SQS / Lambda）

#### 対象コンポーネント
- S3（ARTIFACTS_BUFFER_BUCKET）
- SQS（upload 完了通知）
- Lambda（kcs-edge）

#### 操作の入り口（S3）
1. S3 を開く
2. ARTIFACTS_BUFFER_BUCKET を選択する
3. `uploads/` 配下を確認する
4. 更新時刻で並べ替え、古いファイルが残っていないか確認する

#### 確認する指標
- ファイル数
- 更新時刻
- 数時間以上前のファイルが継続的に残っていないか

判断の目安：
- 数時間以上前のファイルが継続的に残留
    → 完了処理未実行の可能性

---

#### 操作の入り口（SQS）
1. Amazon SQS を開く
2. 対象キューを選択する
3. 「Monitoring」を開く

#### 確認する指標
- ApproximateNumberOfMessages

#### 確認する時間範囲
- 直近 1 時間
    必要に応じて数時間に拡張し、継続的な滞留かを確認する

判断の目安：
- メッセージが継続的に滞留
    → 後続処理異常の可能性

---

### A.5 onprem backend 状態確認
（VPN 経由）

#### 対象コンポーネント
- onprem backend ホスト
- backend コンテナ
- `/srv/kevlumos/uploads`

#### 使用するツール
- SSH
- docker / docker compose

#### 操作の入り口（ログ）
1. onprem backend ホストにログインする
2. backend コンテナのログを確認する

#### 確認コマンド例
- `docker logs <container>`

判断の目安：
- 同一エラーが短時間に連続出力
    → backend 処理異常の可能性

---

#### 操作の入り口（ディスク使用量）
1. `/srv/kevlumos/uploads` の使用量を確認する
2. 全体ディスク空き容量を確認する

#### 確認コマンド例
- `du -sh /srv/kevlumos/uploads`
- `df -h`

判断の目安：
- ディスク容量が急激に増加
    → 一時ファイル滞留の可能性

---

### A.6 cloud / onprem 間通信の確認
（internal 経路）

#### 対象コンポーネント
- Lambda（kcs-edge）
- onprem backend

#### 操作の入り口
- Lambda：CloudWatch Logs
- backend：コンテナログ

#### 確認観点
- Lambda 送信ログと backend 受信ログの対応
- internal header による拒否の有無
- 通信失敗が一時的か継続的か

#### 確認する時間範囲
- 同一エラーが 15 分以上継続しているかを確認する

#### 判断の目安
- 送信あり・受信なし
    → VPN / 経路問題の可能性
- 拒否ログが継続
    → internal 設定不整合の可能性


