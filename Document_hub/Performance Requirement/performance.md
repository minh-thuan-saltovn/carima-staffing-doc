# Performance Requirement — CARIMA Staffing

## 1. 目的

CARIMA Staffing の性能要件は、派遣業務における主要機能である派遣照会、見積、契約、スタッフ管理、勤怠、請求、通知、外部API連携を安定して処理できることを目的とする。

本システムは、企業ユーザーが日常業務で利用する SaaS 型業務システムであるため、通常業務に支障が出ないレスポンス時間を確保するとともに、テナント数、派遣先、派遣元、スタッフ数、取引データ量の増加に応じて段階的に拡張できる性能設計とする。

---

## 2. システム負荷想定

| 項目 | Phase 1 | Phase 2 | Phase 3 |
| --- | --- | --- | --- |
| テナント数 | 5〜10 tenants | 30〜50 tenants | 100+ tenants |
| 登録ユーザー数 | 1,000〜3,000 users | 10,000〜30,000 users | 100,000+ users |
| 同時接続ユーザー数 | 100〜300 users | 1,000〜3,000 users | 10,000+ users |
| スタッフデータ | 10,000 records | 100,000 records | 1,000,000+ records |
| 契約データ | 10,000 records | 100,000 records | 1,000,000+ records |
| 勤怠データ / 月 | 100,000 records | 1,000,000 records | 10,000,000+ records |

---

## 3. レスポンスタイム要件

### 3.1 Web画面レスポンス

| 機能領域 | 目標レスポンスタイム | 備考 |
| --- | --- | --- |
| ログイン / ログアウト | 2秒以内 | 通常時 |
| ダッシュボード表示 | 3秒以内 | サマリーデータを含む |
| 一覧画面検索 | 3秒以内 | 1,000件程度の検索条件を想定 |
| 詳細画面表示 | 2秒以内 | 派遣照会、契約、スタッフ、勤怠 |
| 登録 / 更新処理 | 3秒以内 | 通常トランザクション |
| CSV / Excel ダウンロード要求 | 5秒以内 | ダウンロード開始または非同期処理受付 |
| ファイルアップロード | 10秒以内 | 定義された上限サイズ内 |
| マスタデータ検索 | 2秒以内 | 業務マスタ、権限マスタ |

### 3.2 APIレスポンス

| API種別 | 目標レスポンスタイム | 備考 |
| --- | --- | --- |
| 認証API | 1秒以内 | Token発行 / Refresh |
| マスタ参照API | 1秒以内 | 必要に応じてキャッシュ利用 |
| 業務トランザクションAPI | 2秒以内 | 派遣照会、契約、スタッフ |
| 勤怠入力API | 1.5秒以内 | Staff Mobile App |
| 承認API | 2秒以内 | 勤怠承認、請求承認 |
| 通知トリガーAPI | 2秒以内 | Email / SMS / In-app queue |
| 外部連携API | 5秒以内 | 外部サービスの応答に依存 |

---

## 4. スループット要件

| 処理 | Phase 1 目標 | Phase 2 目標 | Phase 3 目標 |
| --- | --- | --- | --- |
| ログインリクエスト | 10 req/sec | 100 req/sec | 500 req/sec |
| 一般APIリクエスト | 50 req/sec | 500 req/sec | 2,000 req/sec |
| 勤怠登録 | 30 req/sec | 300 req/sec | 1,000 req/sec |
| 検索リクエスト | 20 req/sec | 200 req/sec | 1,000 req/sec |
| 通知ジョブ | 1,000 msg/hour | 50,000 msg/hour | 500,000 msg/hour |
| CSVインポート | 10,000 records/file | 100,000 records/file | 1,000,000 records/file |
| バッチ処理 | 業務SLA内 | 業務SLA内 | 分散ジョブで拡張可能 |

---

## 5. バッチ性能要件

| バッチ処理 | 目標完了時間 | 備考 |
| --- | --- | --- |
| 日次勤怠集計 | 30分以内 | テナント単位 |
| 月次勤怠締め処理 | 2時間以内 | テナント単位 |
| 請求計算処理 | 2時間以内 | 請求サイクル単位 |
| 契約ステータス更新 | 30分以内 | スケジュール実行 |
| 通知リマインド処理 | 15分以内 | 勤務開始前 / 期限前 |
| 監査ログアーカイブ | 1時間以内 | オフピーク時間帯に実行 |
| データバックアップ | バックアップウィンドウ内 | バックアップ要件に準拠 |

---

## 6. データ量要件

| データ種別 | 保持期間 / データ量想定 |
| --- | --- |
| 監査ログ | 最低1年、推奨3〜7年 |
| 勤怠データ | 最低5年 |
| 契約データ | 最低5〜7年 |
| 請求データ | 最低7年 |
| 通知履歴 | 最低1〜3年 |
| アップロード契約ファイル | テナント契約ポリシーに準拠 |
| スタッフ個人情報 | 個人情報保護および法令上の保持要件に準拠 |

---

## 7. 検索性能要件

検索画面では、tenant_id、会社、支店、ステータス、日付範囲、スタッフ、契約、承認状態などの条件による実務的な検索を可能とする。

| 検索画面 | 目標レスポンスタイム |
| --- | --- |
| 派遣照会検索 | 3秒以内 |
| 見積依頼検索 | 3秒以内 |
| 契約検索 | 3秒以内 |
| スタッフ検索 | 3秒以内 |
| 勤怠検索 | 5秒以内 |
| 請求検索 | 5秒以内 |
| 監査ログ検索 | 10秒以内 |

大量データを扱う画面では、ページング、インデックス最適化、非同期エクスポート、検索条件の制限を適用する。

---

## 8. ファイルインポート / エクスポート要件

| 機能 | 要件 |
| --- | --- |
| CSVインポート | 事前バリデーション後にデータ登録を行う |
| CSVエクスポート | 大量データの場合はバックグラウンドジョブで実行する |
| Excelダウンロード | テンプレートベースのダウンロードに対応する |
| 契約ファイルアップロード | 進捗表示およびエラーハンドリングを行う |
| 大容量ファイル処理 | 通常画面操作に影響を与えない設計とする |

推奨上限値：

| 項目 | 上限 |
| --- | --- |
| 通常アップロードファイルサイズ | 10〜50 MB |
| CSVインポート行数 | Phase 1: 10,000 rows |
| Excelエクスポート行数 | Phase 1: 50,000 rows |
| バックグラウンドエクスポート行数 | 50,000+ rows |

---

## 9. Mobile App 性能要件

Staff Mobile App では、スタッフによる勤怠入力を最優先業務とし、シンプルかつ高速な操作性を確保する。

| 機能 | 目標レスポンスタイム |
| --- | --- |
| アプリログイン | 2秒以内 |
| 出勤 / 退勤登録 | 1.5秒以内 |
| 勤怠修正申請 | 2秒以内 |
| シフト / 契約確認 | 2秒以内 |
| 通知一覧表示 | 2秒以内 |

通信環境が不安定な場合は、明確なリトライメッセージを表示する。オフライン対応は Phase 2 または Phase 3 の拡張機能として検討する。

---

## 10. 高負荷時の可用性

本システムは、通常業務のピーク時間帯においても安定稼働できることを前提とする。

想定されるピークタイミング：

- 朝の勤怠入力時間帯
- 月末の勤怠締め処理
- 請求計算期間
- 契約更新期間
- 一括通知送信時
- CSVインポート / エクスポート実行時

請求計算、CSVエクスポート、通知送信、ログアーカイブなどの重い処理は、オンライン画面のレスポンスに影響を与えないよう非同期処理とする。

---

## 11. 性能設計方針

上記要件を満たすため、以下の設計方針を適用する。

- tenant_id、status、date、company_id、staff_id、contract_id に対するDBインデックス設計
- すべての一覧画面にページングを適用
- 重い処理は非同期ジョブキューで実行
- マスタデータおよび共通参照データは必要に応じてキャッシュ化
- Phase 2以降では必要に応じて Read / Write 分離を検討
- アップロードファイルはオブジェクトストレージに保存
- APIレイテンシ、DBスロークエリ、Queue遅延、Batch実行時間を監視
- 本番リリース前に負荷試験を実施
- 大規模運用に備え、テナント単位でのデータ分割方針を検討

---

## 12. Phase別定義

### Phase 1: Production Minimum

Phase 1 は、限定されたテナント数で本番運用を開始するための最低限の性能要件を対象とする。

主な要件：

- 通常画面レスポンスは 2〜5秒以内
- 基本APIレスポンスは 1〜3秒以内
- バッチ処理は日次業務時間内に完了
- CSVインポート / エクスポートは上限件数を定義して対応
- APIレイテンシおよびDBスロークエリの基本監視を実施

### Phase 2: Enterprise Operation

Phase 2 は、エンタープライズ顧客および大規模データ運用を想定した性能強化を対象とする。

主な要件：

- より多い同時接続ユーザーに対応
- 大量データのインポート / エクスポートをバックグラウンドジョブ化
- Queue監視を実施
- Performance Dashboard を整備
- DBインデックスチューニングおよびスロークエリ改善を実施
- 月末ピーク処理を想定した負荷試験を実施

### Phase 3: Large Scale / High Availability

Phase 3 は、大規模テナントおよび高負荷 SaaS 運用を想定した拡張性能要件を対象とする。

主な要件：

- 水平スケーリング対応
- Read Replica / DB Partitioning の検討
- 分散ジョブ処理
- Multi-region または DR-ready 構成
- 大規模負荷試験
- テナントプラン別の Performance SLA 定義

---

# Performance Requirement — CARIMA Staffing (VI)

## 1. Mục tiêu

Performance Requirement của CARIMA Staffing nhằm đảm bảo hệ thống có thể xử lý ổn định các nghiệp vụ chính của dịch vụ phái cử nhân sự, bao gồm 派遣照会, 見積, 契約, スタッフ管理, 勤怠, 請求, 通知 và API連携.

Hệ thống cần đáp ứng tốc độ phản hồi phù hợp cho người dùng doanh nghiệp, đồng thời có khả năng mở rộng theo số lượng tenant, 派遣先, 派遣元, staff và dữ liệu giao dịch tăng dần.

---

## 2. Giả định tải hệ thống

| Item | Phase 1 | Phase 2 | Phase 3 |
| --- | --- | --- | --- |
| Tenant số lượng | 5〜10 tenants | 30〜50 tenants | 100+ tenants |
| User đăng ký | 1,000〜3,000 users | 10,000〜30,000 users | 100,000+ users |
| Concurrent user | 100〜300 users | 1,000〜3,000 users | 10,000+ users |
| Staff data | 10,000 records | 100,000 records | 1,000,000+ records |
| Contract data | 10,000 records | 100,000 records | 1,000,000+ records |
| Attendance records / month | 100,000 records | 1,000,000 records | 10,000,000+ records |

---

## 3. Response Time Requirement

### 3.1 Web Screen Response

| Function Area | Target Response Time | Note |
| --- | --- | --- |
| Login / Logout | ≤ 2 sec | 通常時 |
| Dashboard display | ≤ 3 sec | Summary data included |
| List screen search | ≤ 3 sec | 1,000 records search condition |
| Detail screen display | ≤ 2 sec | 派遣照会, 契約, スタッフ, 勤怠 |
| Create / Update screen save | ≤ 3 sec | Normal transaction |
| CSV / Excel download request | ≤ 5 sec | Start download or accept background job |
| File upload | ≤ 10 sec | File size under defined limit |
| Master data search | ≤ 2 sec | 業務マスタ, 権限マスタ |

### 3.2 API Response

| API Type | Target Response Time | Note |
| --- | --- | --- |
| Authentication API | ≤ 1 sec | Token issue / refresh |
| Master reference API | ≤ 1 sec | Cached where possible |
| Business transaction API | ≤ 2 sec | 派遣照会, 契約, スタッフ |
| Attendance input API | ≤ 1.5 sec | Staff mobile app |
| Approval API | ≤ 2 sec | 勤怠承認, 請求承認 |
| Notification trigger API | ≤ 2 sec | Email / SMS / in-app queue |
| External integration API | ≤ 5 sec | Depends on external service |

---

## 4. Throughput Requirement

| Process | Phase 1 Target | Phase 2 Target | Phase 3 Target |
| --- | --- | --- | --- |
| Login request | 10 req/sec | 100 req/sec | 500 req/sec |
| General API request | 50 req/sec | 500 req/sec | 2,000 req/sec |
| Attendance submission | 30 req/sec | 300 req/sec | 1,000 req/sec |
| Search request | 20 req/sec | 200 req/sec | 1,000 req/sec |
| Notification job | 1,000 msg/hour | 50,000 msg/hour | 500,000 msg/hour |
| CSV import | 10,000 records/file | 100,000 records/file | 1,000,000 records/file |
| Batch processing | Within business SLA | Within business SLA | Scalable distributed job |

---

## 5. Batch Performance Requirement

| Batch Process | Target Completion Time | Note |
| --- | --- | --- |
| Daily attendance aggregation | ≤ 30 min | Tenant-level batch |
| Monthly attendance closing | ≤ 2 hours | Per tenant |
| Invoice calculation | ≤ 2 hours | Per billing cycle |
| Contract status update | ≤ 30 min | Scheduled batch |
| Notification reminder batch | ≤ 15 min | Before work start / deadline |
| Audit log archive | ≤ 1 hour | Off-peak execution |
| Data backup | Within backup window | Follow backup requirement |

---

## 6. Data Volume Requirement

| Data Type | Retention / Volume Assumption |
| --- | --- |
| Audit log | Minimum 1 year, recommended 3〜7 years depending on tenant policy |
| Attendance data | Minimum 5 years |
| Contract data | Minimum 5〜7 years |
| Invoice data | Minimum 7 years |
| Notification history | Minimum 1〜3 years |
| Uploaded contract files | Depends on tenant contract policy |
| Staff personal data | Follow privacy and legal retention rules |

---

## 7. Search Performance Requirement

Search screens must support practical filtering by tenant, company, branch, status, date range, staff, contract, and approval state.

| Search Screen | Target |
| --- | --- |
| 派遣照会検索 | ≤ 3 sec |
| 見積依頼検索 | ≤ 3 sec |
| 契約検索 | ≤ 3 sec |
| スタッフ検索 | ≤ 3 sec |
| 勤怠検索 | ≤ 5 sec |
| 請求検索 | ≤ 5 sec |
| Audit log search | ≤ 10 sec |

For large data screens, pagination, index optimization, asynchronous export, and search condition limitation must be applied.

---

## 8. File Import / Export Requirement

| Function | Requirement |
| --- | --- |
| CSV import | Validate first, then register data |
| CSV export | For large data, execute as background job |
| Excel download | Template-based download supported |
| Contract file upload | Upload progress and error handling required |
| Large file process | Should not block normal screen operation |

Recommended limits:

| Item | Limit |
| --- | --- |
| Normal upload file size | 10〜50 MB |
| CSV import rows | Phase 1: 10,000 rows |
| Excel export rows | Phase 1: 50,000 rows |
| Background export rows | 50,000+ rows |

---

## 9. Mobile App Performance Requirement

Staff Mobile App must prioritize simple and fast operations for attendance input.

| Function | Target Response Time |
| --- | --- |
| App login | ≤ 2 sec |
| Attendance clock-in / clock-out | ≤ 1.5 sec |
| Attendance correction request | ≤ 2 sec |
| Shift / contract confirmation | ≤ 2 sec |
| Notification list | ≤ 2 sec |

In unstable network conditions, the app should show clear retry messages. Offline support can be considered as Phase 2 or Phase 3 option.

---

## 10. Availability Under Load

The system must continue stable operation under normal business peak load.

Peak timing examples:

- Morning attendance input
- End-of-month attendance closing
- Invoice calculation period
- Contract renewal period
- Mass notification period
- CSV import / export operation

Heavy jobs such as invoice calculation, CSV export, notification sending, and log archive should be executed asynchronously to avoid affecting online screen performance.

---

## 11. Performance Design Policy

To achieve the above requirements, the system should apply the following design principles:

- Database indexing for tenant_id, status, date, company_id, staff_id, contract_id
- Pagination for all list screens
- Async job queue for heavy processing
- Cache for master data and common reference data
- Read/write separation if required in Phase 2+
- Object storage for uploaded files
- Monitoring for API latency, DB slow query, queue delay, and batch execution time
- Load testing before production release
- Tenant-level data partitioning strategy for large-scale operation

---

## 12. Phase Definition

### Phase 1: Production Minimum

Phase 1 focuses on stable production release for limited tenants.

Performance target is basic business operation without major delay.

Main requirements:

- Normal screen response within 2〜5 seconds
- Basic API response within 1〜3 seconds
- Batch completed within daily operation window
- CSV import/export with defined upper limit
- Basic monitoring for API latency and DB slow query

### Phase 2: Enterprise Operation

Phase 2 targets enterprise customers and larger data volume.

Main requirements:

- Higher concurrent user support
- Background job processing for heavy export/import
- Queue monitoring
- Performance dashboard
- DB index tuning and slow query improvement
- Load test for monthly peak operation

### Phase 3: Large Scale / High Availability

Phase 3 targets large tenants and high-volume SaaS operation.

Main requirements:

- Horizontal scaling
- Read replica / DB partitioning
- Distributed job processing
- Multi-region or DR-ready design
- Large-scale load testing
- Performance SLA definition per tenant plan