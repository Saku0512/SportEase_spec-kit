設計書概要SportEaseは仙台高専向けに設計されたマイクロサービスベースのスポーツイベント管理システムです。システムは、APIバックエンド、管理者フロントエンド、学生フロントエンド用の別々のコンテナ化されたサービスを持つ3層アーキテクチャに従います。設計は、Google OAuth統合、役割ベースアクセス制御、QRコードベースの参加確認を通じてセキュリティを重視しています。アーキテクチャシステムアーキテクチャgraph TB
    subgraph "Client Layer"
        A[SportEase-Admin<br/>Next.js]
        B[SportEase-Student<br/>Next.js]
    end

    subgraph "API Layer"
        C[SportEase-API<br/>Go/Gin]
    end

    subgraph "Data Layer"
        D[MySQL Database]
        E[Redis Cache]
    end

    subgraph "External Services"
        F[Google Cloud Identity]
    end

    A --> C
    B --> C
    C --> D
    C --> E
    A --> F
    B --> F
コンテナアーキテクチャgraph LR
    subgraph "Docker Compose Environment"
        subgraph "Frontend Services"
            A[sportease-admin:3000]
            B[sportease-student:3001]
        end

        subgraph "Backend Services"
            C[sportease-api:8080]
        end

        subgraph "Data Services"
            D[mysql:3306]
            E[redis:6379]
        end

        F[nginx:80] --> A
        F --> B
        A --> C
        B --> C
        C --> D
        C --> E
    end
コンポーネントとインターフェース1. SportEase-API（バックエンド）技術スタック:Go 1.21+ with Gin frameworkGORM for database ORMJWT for session managementGoogle API Client Library for Go for authenticationコアモジュール:認証モジュールドメイン制限付きGoogle OAuth 2.0統合JWTトークン生成と検証役割ベースアクセス制御ミドルウェアRedisによるセッション管理ユーザー管理モジュール役割割り当て付きユーザーCRUD操作CSV一括インポート機能ユーザー役割に基づく権限検証すべてのユーザー操作の監査ログイベント管理モジュール競技作成と管理チーム登録と検証トーナメント表生成（ランダムシード）試合スケジュールと結果追跡QRコードモジュール時間ベース検証付き動的QRコード生成参加ログ付きQRコード確認出席記録管理不正防止検証（時間窓、ユーザー適格性）通知モジュールお知らせ作成と配信試合結果のリアルタイム通知クラス固有およびグローバルメッセージング重要な更新のメール統合2. SportEase-Admin（管理者フロントエンド）技術スタック:Next.js 14 with App RouterTypeScript for type safetyTailwind CSS for stylingReact Query for state managementChart.js for dashboard visualizations主要機能:ダッシュボードリアルタイム競技進行概要アクティブな試合と今後のイベントシステムヘルスとユーザーアクティビティメトリクス一般的なタスクのクイックアクションボタンユーザー管理インターフェース役割割り当て付きユーザー作成/編集フォーム検証フィードバック付きCSVアップロードインターフェース役割管理用権限マトリックスユーザーアクティビティとログイン履歴ビュー競技管理インターフェース検証付き競技作成ウィザードドラッグアンドドロップ付きトーナメント表エディタースコア検証付き試合結果入力フォームトーナメント表のPDFエクスポート機能QRコード確認インターフェースデバイスカメラを使用したQRコードスキャナー統合リアルタイム検証フィードバック参加履歴と分析大規模イベント用一括確認3. SportEase-Student（学生フロントエンド）技術スタック:Next.js 14 with App RouterTypeScript for type safetyTailwind CSS for stylingPWA capabilities for mobile experienceQR code generation library主要機能:個人ダッシュボードクラスとチーム情報付きユーザープロフィール今後の競技とスケジュール個人参加履歴未読インジケーター付き通知センター競技ビューアー競技詳細とルールリアルタイムトーナメント表試合結果と順位チームメンバー情報QRコードジェネレーター時間制限付きQRコード生成検証付き競技固有コードコード状態の視覚的フィードバック生成されたコードのオフライン機能データモデルコアエンティティユーザーモデルtype User struct {
    ID             uint      `gorm:"primaryKey"`
    GoogleID       string    `gorm:"uniqueIndex;not null"` // MicrosoftOIDから変更
    Email          string    `gorm:"uniqueIndex;not null"`
    DisplayName    string    `gorm:"not null"`
    ClassID        *uint     `gorm:"index"`
    IsInitialLogin bool      `gorm:"default:true"`
    Class          Class     `gorm:"foreignKey:ClassID"`
    Roles          []Role    `gorm:"many2many:user_roles"`
    CreatedAt      time.Time
    UpdatedAt      time.Time
}
イベントモデルtype Event struct {
    ID                      uint      `gorm:"primaryKey"`
    Name                    string    `gorm:"not null"`
    Description             string    `gorm:"type:text"`
    Location                string
    StartAt                 *time.Time
    MinParticipantsPerClass *uint
    MaxParticipantsPerClass *uint
    IsExperienceRestricted  bool      `gorm:"default:false"`
    Teams                   []Team    `gorm:"foreignKey:EventID"`
    Matches                 []Match   `gorm:"foreignKey:EventID"`
    CreatedAt               time.Time
    UpdatedAt               time.Time
}
QRコードデータ構造type QRCodeData struct {
    UserID      uint      `json:"user_id"`
    EventID     uint      `json:"event_id"`
    GeneratedAt time.Time `json:"generated_at"`
    ExpiresAt   time.Time `json:"expires_at"`
    Signature   string    `json:"signature"`
}
データベース関係erDiagram
    USERS ||--o{ USER_ROLES : has
    USER_ROLES }o--|| ROLES : references
    USERS }o--|| CLASSES : belongs_to
    CLASSES ||--o{ TEAMS : has
    TEAMS }o--|| EVENTS : participates_in
    TEAMS ||--o{ TEAM_MEMBERS : contains
    TEAM_MEMBERS }o--|| USERS : references
    EVENTS ||--o{ MATCHES : contains
    MATCHES ||--o{ MATCH_PARTICIPANTS : has
    MATCH_PARTICIPANTS }o--|| TEAMS : references
    USERS ||--o{ ATTENDANCE_RECORDS : creates
    ATTENDANCE_RECORDS }o--|| EVENTS : for
エラーハンドリングAPIエラーレスポンス形式{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid input data",
    "details": {
      "field": "email",
      "reason": "Domain not allowed"
    },
    "timestamp": "2024-01-15T10:30:00Z"
  }
}
エラーカテゴリ認証エラー: 無効なトークン、ドメイン制限認可エラー: 権限不足、役割違反検証エラー: 無効な入力データ、ビジネスルール違反リソースエラー: 見つからない、競合、依存関係の問題システムエラー: データベース障害、外部サービス利用不可エラーハンドリング戦略APIでの集中エラーミドルウェアすべてのエンドポイントでの一貫したエラーレスポンス形式ユーザーフレンドリーなメッセージ付きクライアントサイドエラー境界相関IDを使用した包括的ログ記録非重要機能の優雅な劣化テスト戦略バックエンドテスト単体テスト: 80%以上のカバレッジでの個別関数・メソッドテスト統合テスト: データベース操作と外部サービス相互作用APIテスト: 様々な入力シナリオでのエンドポイントテストセキュリティテスト: 認証、認可、入力検証フロントエンドテストコンポーネントテスト: Reactコンポーネントの動作とレンダリング統合テスト: ユーザーワークフローとAPI相互作用E2Eテスト: Playwrightを使用した重要なユーザージャーニーアクセシビリティテスト: WCAG準拠検証QRコードテスト生成テスト: 適切な有効期限での有効なコード作成確認テスト: 様々なシナリオでのコード検証セキュリティテスト: 改ざん検出とリプレイ攻撃防止パフォーマンステスト: 大量コード生成と確認テストデータ管理自動テストデータベースシードフィーチャーブランチごとの分離されたテスト環境外部サービスのモック（Google People API）テストデータクリーンアップとリセット手順セキュリティ考慮事項認証セキュリティPKCEフロー付きGoogle OAuth 2.0APIレベルでのドメイン制限強制JWTトークンローテーションと安全な保存セッションタイムアウトと自動ログアウト認可セキュリティ最小権限の原則による役割ベースアクセス制御リソースレベル権限検証管理者アクション監査ログクラス管理者のクラスレベルデータ分離QRコードセキュリティ時間ベースコード有効期限（10分間窓）暗号署名検証リプレイ攻撃防止ユーザー適格性確認データセキュリティデータベース接続暗号化（TLS）保存時の機密データ暗号化入力検証とサニタイゼーションORMによるSQLインジェクション防止インフラストラクチャセキュリティコンテナイメージ脆弱性スキャンサービス間ネットワークセグメンテーションシークレット用環境変数管理定期的なセキュリティ更新とパッチパフォーマンス考慮事項データベース最適化頻繁にクエリされる列への適切なインデックス設定可能な制限付きコネクションプーリング複雑なトーナメント操作のクエリ最適化スキーマ変更のデータベースマイグレーション戦略キャッシュ戦略頻繁にアクセスされるデータのRedisキャッシュアクティブな競技中のトーナメント表キャッシュ認証用ユーザーセッションキャッシュ静的コンテンツのAPIレスポンスキャッシュフロントエンドパフォーマンスコード分割と遅延読み込み画像最適化と圧縮プログレッシブWebアプリ機能重要機能のオフライン機能スケーラビリティ計画APIサービスの水平スケーリング機能ロードバランシング設定データベース読み取りレプリカサポート静的アセットのCDN統合
