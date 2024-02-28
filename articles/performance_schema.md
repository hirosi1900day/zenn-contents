---
title: "performance_schema を効果的な活用方法"
emoji: "🗂"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [mysql]
published: true
---
## performance_schemaとは
performance_schema は MySQL と Aurora MySQL の両方で使用可能な機能で、データベースの実行時のパフォーマンスやリソース消費に関する詳細な情報を収集します。(aws rdsなどだとPerformance Insightsで確認可能になっている)

## Performance Insights で収集可能な情報
- 1. パフォーマンス監視
待機イベントの分析: performance_schemaの待機イベントテーブルを使用して、データベースの待機時間を分析します。これにより、データベースがリソース（CPU、IO、ネットワーク等）を待機している時間とその原因を特定できます。

ステートメント分析: SQLクエリのパフォーマンスを分析するために、実行されたステートメントの統計情報を提供するテーブルを使用します。これにより、最もリソースを消費しているクエリや最も頻繁に実行されるクエリを特定できます。

- 2. パフォーマンスチューニング
スロークエリの特定: 遅いクエリを特定し、クエリの最適化を行います。performance_schemaでは、実行に時間がかかったクエリや特定の条件を満たすクエリを見つけることができます。

インデックス使用状況の分析: テーブルのインデックス使用状況を分析し、不要なインデックスの削除や必要なインデックスの追加を行います。これにより、クエリのパフォーマンスを向上させることができます。

- 3. 監視とアラート
カスタムモニタリングツールの統合: performance_schemaから収集されるデータを使用して、カスタム監視ダッシュボードを作成します。また、特定のメトリクスが閾値を超えた場合にアラートを発するように設定することができます。

- 4. ロックとデッドロックの分析
ロック待機分析: テーブルレベルや行レベルのロック待機情報を分析し、ロック競合がパフォーマンスにどのように影響しているかを理解します。

デッドロックの特定: デッドロックの発生を検出し、その原因を特定するための情報を提供します。デッドロックのログを解析して、問題のあるトランザクションを特定します。

## performance_schema を効果的に使用するための具体的なコマンドやクエリ例
- 1. performance_schemaの有効化
MySQLでperformance_schemaが有効になっていることを確認します。デフォルトで有効になっていることが多いですが、無効になっている場合は、MySQLの設定ファイル（my.cnfまたはmy.ini）に以下の行を追加して有効化します。

```
[mysqld]
performance_schema=ON
MySQLを再起動後、以下のコマンドでperformance_schemaが有効になっているかを確認できます。
```

```sql
SHOW VARIABLES LIKE 'performance_schema';
```

- 2. ステートメント分析
最もリソースを消費しているSQLステートメントを特定します。以下のクエリは、CPU時間、待機時間、ロック時間、およびIO時間で最もコストが高いステートメントを見つけます。

```sql
SELECT EVENT_NAME, COUNT_STAR, SUM_TIMER_WAIT 
FROM performance_schema.events_statements_summary_global_by_event_name 
WHERE SUM_TIMER_WAIT > 0 
ORDER BY SUM_TIMER_WAIT DESC 
LIMIT 10;
```

- 3. 待機イベント分析
データベースの待機イベントを分析して、どのリソースがボトルネックになっているかを特定します。

```sql
SELECT * FROM performance_schema.events_waits_summary_global_by_event_name 
WHERE COUNT_STAR > 0 
ORDER BY SUM_TIMER_WAIT DESC 
LIMIT 10;
```

4. スロークエリの識別
performance_schemaを使用して、実行時間が長いクエリを特定します。events_statements_history_longテーブルには、最も長い実行時間を持つステートメントの履歴が保持されています。

```sql
SELECT SQL_TEXT, TIMER_WAIT 
FROM performance_schema.events_statements_history_long 
ORDER BY TIMER_WAIT DESC 
LIMIT 10;
```
- 5. ロック待機分析
ロック待機の原因となっているクエリを特定します。

```sql
SELECT * FROM performance_schema.events_waits_current 
WHERE EVENT_NAME LIKE '%lock%' 
ORDER BY EVENT_ID;
```