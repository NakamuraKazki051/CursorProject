# CursorProject

Cursor（と Claude Code）で作った3つの Web アプリをまとめたフォルダです。

各アプリのフォルダは、それぞれ別の GitHub リポジトリです。このリポジトリでは管理していません（`.gitignore` で外しています）。

## 3つのアプリをひと目で

| | タスク管理 | 収支管理 | RaiseLine |
|---|---|---|---|
| どんなアプリ | やることをカンバンで管理 | お金の出入りをカンバンで記録（袋分け家計簿） | 旧Twitter風の SNS |
| フォルダ | `TaskManagement/` | `IncomeandExpenceManagement/` | `RaiseLine/` |
| リポジトリ | [task-manager-app](https://github.com/NakamuraKazki051/task-manager-app) | [finance-manager-app](https://github.com/NakamuraKazki051/finance-manager-app) | [sns-app](https://github.com/NakamuraKazki051/sns-app) |
| 画面 | HTML / CSS / JavaScript（フレームワークなし） | Next.js 16 + React 19 + TypeScript | React 19 + TypeScript + Vite |
| API | Java 25 + Spring Boot 4.1 | Ruby 4.0 + Rails 8.1 | Java 25 + Spring Boot 4.1 |
| データベース | PostgreSQL 16 | MySQL 8.4 | MySQL 8.4 |
| ローカルで開く URL | http://localhost:8080/ | http://localhost:8080/ | http://localhost:5173/（開発中）・http://localhost:8080/ |
| AWS | EC2 + RDS（Terraform） | EC2 + RDS（Terraform） | CloudFront + EC2 + RDS + S3（Terraform） |

3つとも、ブラウザから Cookie でログインし、データはサーバーの DB に保存します。ローカルでは DB を Docker で動かし、AWS には使うときだけ環境を作って、使い終わったら消します。

作った順番：**タスク管理 → 収支管理（タスク管理をもとに作った） → RaiseLine**

---

## 1. タスク管理アプリ（`TaskManagement/`）

**ひとことで：** 付せんをボードに貼って動かすように、タスクを「未着手 → 進行中 → 完了」の列へドラッグして進めるアプリです。

### できること
- **ボードと列：** ボードを用途ごとに何枚でも作れます。列の追加・名前変更・並べ替えもでき、列ごとに「完了扱い」を決められます。
- **タスク：**
  - 期限・優先度（高・中・低）・カテゴリ（色分けのタグ）・チェックリストを付けられます。
  - 期限が近いと「今日まで」「明日まで」と出し、過ぎると強調します。
- **探す・並べる：**
  - キーワード検索と、カテゴリ・優先度での絞り込みができます。
  - 並び順は、手動・期限順・優先度順から選べます。
- **まとめて操作：** 選択モードで、タスクをまとめて削除できます。
- **知らせる：** 期限当日や期限切れのタスクがあると、ブラウザの通知が1日1回届きます。
- **その他：**
  - JSON でのエクスポート・インポート（バックアップ）
  - ダークテーマ、キーボード操作（`N` で追加、`/` で検索）、スマホ表示

### 動かし方
```
docker compose up -d          # PostgreSQL
cd backend; mvn package
java -jar target/task-manager-api-0.1.0.jar   # http://localhost:8080/ を開く
```
くわしくは [`TaskManagement/README.md`](./TaskManagement/README.md)。

---

## 2. 収支管理アプリ（`IncomeandExpenceManagement/`）

**ひとことで：** 列＝カテゴリ（給与・食費など）、カードは1件のお金の出入りです。月ごとの収入・支出・残高がひと目でわかる家計簿です。

タスク管理アプリをもとに作りました。ログインやカンバンの操作はそのまま引き継ぎ、画面は Next.js、API は Rails で作り直しています。

### できること
- **取引の記録：**
  - 金額・日付・カテゴリ・メモ・タグを付けて登録します。
  - 合計はサーバーが円の整数で計算するので、端数のずれが出ません。
- **月ごとの集計：**
  - 月を切り替えると、その月の収入・支出・残高が出ます。
  - カテゴリごとの合計は、列の見出しに出ます。
- **予定と実績：**
  - 来月の給料のようなまだ確定していないお金は「予定」（点線のカード）にします。
  - 予定は合計に入れず、別に表示します。
- **固定費：** 家賃のような毎月の出費を登録しておくと、毎月の取引が自動で作られます（🔁 の印が付きます）。
- **予算：** 支出カテゴリごとに月の予算を決めると、残りの額と使った割合をバーで出し、超えたら赤くします。
- **レポート：** カテゴリ別の内訳と、12か月の収入・支出の推移をグラフで出します。
- **その他：**
  - CSV 出力（Excel でも文字化けしない）
  - JSON のエクスポート・インポート
  - 一覧表示とカンバン表示の切り替え、ダークテーマ、スマホ表示

### 動かし方
```
docker compose up -d                          # MySQL
cd frontend; npm install; npm run build:rails  # 画面を書き出す
cd ../backend; bundle install
ruby bin/rails db:prepare
ruby bin/rails server -p 8080                  # http://localhost:8080/ を開く
```
くわしくは [`IncomeandExpenceManagement/README.md`](./IncomeandExpenceManagement/README.md)。

---

## 3. RaiseLine（`RaiseLine/`）

**ひとことで：** 140文字の投稿に画像を1枚付けて、タイムラインで読み、リプライ・いいね・リポスト・フォローで反応できる、旧Twitter風の SNS です。

3つの中で一番大きなアプリです。要件定義書・仕様書・API 仕様書（自動生成）・結合テスト・性能テストまでそろえています。

### できること
- **投稿：**
  - 140文字の投稿に、画像を1枚付けられます。投稿の編集・削除もできます。
  - 「#タグ」はリンクになり、よく使われているタグはトレンドに出ます。
- **タイムライン：**
  - 「ホーム（フォロー中）」「みんな」「おすすめ（よく反応された投稿）」の3つのタブがあります。
  - 新しい投稿は「↑ 新しい投稿 N件」で知らせます。
- **反応：** リプライ、いいね、リポスト、フォローができます。
- **人を探す：** ユーザー検索と「おすすめのユーザー」があります。
- **通知と DM：**
  - いいねやフォローの通知が届きます。
  - DM は1対1で、画像の添付や送信の取り消しができます。
  - どちらも WebSocket ですぐに届きます。
- **安心して使うために：**
  - ブロック・ミュート
  - 鍵アカウント（承認したフォロワーだけが投稿を見られる）
  - 通報と、管理者がそれを処理する画面
- **アカウント：**
  - 新規登録のメール確認と、パスワードの再設定（メール）
  - 2段階認証（認証アプリ）
  - ログイン中の端末の一覧と、端末ごとのログアウト
  - アカウントの削除

### 動かし方
```
docker compose up -d                    # MySQL（ポート 3307）
cd backend; mvn spring-boot:run         # API（http://localhost:8080）
cd frontend; npm install; npm run dev   # 画面（http://localhost:5173 を開く）
```
くわしくは [`RaiseLine/README.md`](./RaiseLine/README.md) と [`RaiseLine/docs/`](./RaiseLine/docs/)（要件定義書・仕様書）。

---

## 3つに共通すること

- **テスト：**
  - どのアプリも API の自動テストがあります（タスク管理は JUnit、収支管理は Rails のテスト）。
  - RaiseLine は画面のテスト（Vitest）と、Playwright による結合テストもあります。PR を出すたびに GitHub Actions がテストを動かし、通ってからマージします。
- **性能テスト：** `perf/` に k6 のシナリオがあり、たくさんのデータを入れて速さを確かめられます。
- **AWS：**
  - `infra/terraform/` に、無料枠の範囲で環境を作る設定があります。
  - 使うときだけ作り、終わったら消して費用を抑えます。
- **このPCで使うときの注意：**
  - メモリが少ないので、Docker Desktop は必要なときだけ起動します。
  - 終わったら `docker compose stop` で DB を止めます（データは残ります）。
