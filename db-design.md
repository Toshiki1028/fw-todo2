# やること管理アプリ DB設計書

## 1. 対象と参照元

保存先はMySQLとし、このアプリが使うテーブル（データを保存する表）は `todos` の1つだけとする。やること1件を1行に保存する。本書は `requirements.md` と `handout_app_spec.md` に基づく。画面の詳細は画面設計書で扱う。

## 2. テーブルとカラムの定義

`NULL` は値を入れない状態、`NOT NULL` は値が必須であることを表す。主キーは各行を重複なく見分ける番号である。

| カラム（保存する項目） | 型 | NULL | 初期値・自動設定 | 対応する要件 |
|---|---|---|---|---|
| `id` | `BIGINT` | 不可 | 主キー。1ずつ自動採番する。 | 1件を番号で見分ける。編集・削除の対象指定と操作記録に使う。要件定義書 3・4・7、叩き台 2・3・5-5。 |
| `title` | VARCHAR`(255)` | 不可 | なし | 必須の「やること」。名前での部分一致検索に使う。255文字以内とし、空白だけはアプリ側で拒否する。要件定義書 3・5・6、叩き台 2・5-1・5-4。 |
| `detail` | `VARCHAR(255)` | 可 | なし | 省略できる「メモ」。255文字以内とする。要件定義書 3・5、叩き台 2・5-1。 |
| `category` | `VARCHAR(255)` | 不可 | なし | 必須の「ジャンル」。指定の5種類から選び、完全一致検索に使う。要件定義書 3・5・6、叩き台 2・5-1・5-4。 |
| `priority` | `INT` | 不可 | `2`（中） | 優先度。`1`＝高、`2`＝中、`3`＝低。要件定義書 3・4・5、叩き台 2・4・5-1。 |
| `due_date` | `DATE` | 可 | なし | 省略できる期限。近い順・遠い順の並び替えに使う。要件定義書 3・6、叩き台 2・5-4。 |
| `completed` | `BOOLEAN` | 不可 | `FALSE`（未完了） | 完了状態。`1`＝完了、`0`＝未完了。編集で変更する。要件定義書 3・4、叩き台 2・4。 |
| `created_at` | `DATETIME` | 不可 | 登録時にMySQLが現在日時を入れる。 | 登録日時を自動保存する。要件定義書 3、叩き台 2。 |
| `updated_at` | `DATETIME` | 不可 | 登録時と更新時にMySQLが現在日時を入れる。 | 最後に直した日時を自動保存する。要件定義書 3、叩き台 2。 |

ジャンルに使える値は「デザイン」「マーケティング」「プログラミング」「資格」「就職活動」の5種類である。`id`・`created_at`・`updated_at` は利用者が入力せず、MySQLが自動で設定する。

## 3. CREATE TABLE文（DDL）

DDLはテーブルの形を定義するSQL（データベースへの命令）である。以下は設計として記載するもので、本書の作成時には実行しない。

```sql
CREATE TABLE todos (
    id BIGINT NOT NULL AUTO_INCREMENT,
    title VARCHAR(255) NOT NULL,
    detail VARCHAR(255) NULL,
    category VARCHAR(255) NOT NULL,
    priority INT NOT NULL DEFAULT 2,
    due_date DATE NULL,
    completed BOOLEAN NOT NULL DEFAULT FALSE,
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    PRIMARY KEY (id)
);
```

`AUTO_INCREMENT` は番号を1ずつ自動で増やす指定、`DEFAULT` は値を指定しなかった場合の初期値、`ON UPDATE CURRENT_TIMESTAMP` は更新時に日時を自動で入れ直す指定である。

## 4. 保存前の確認と検索での利用

- `title` は必須で255文字以内、`detail` は255文字以内とする。半角・全角の空白だけの入力は空として扱う。これらの入力確認は保存前にアプリ側で行う。
- `category` は画面の選択肢に加え、アプリ側でも保存前に5種類のどれかを確認する。画面を経由せずに送られた指定外の値も保存しない。
- `priority` は画面で高・中・低から選ぶ必須項目とする。DBに初期値 `2` があっても、画面で未選択なら入力エラーとする。
- 名前検索は `title` の部分一致、ジャンル検索は `category` の完全一致とする。両方を指定した場合は両条件を満たす行だけを使う。
- 期限順には `due_date` を使う。初期表示は近い順、期限のない行は近い順・遠い順のどちらでも最後にする。
- 登録・編集・削除の記録には対象の `id` を使い、`title` と `detail` の中身は記録しない。記録用のテーブルは設計しない。

## 5. 参照元との対応

| 参照元 | この設計書で反映した内容 |
|---|---|
| `requirements.md` 2・4 | 一覧・登録・編集・削除で使う保存項目と、番号による対象指定。 |
| `requirements.md` 3 ／ `handout_app_spec.md` 2 | `todos` の9カラム、型、必須条件、初期値、自動設定。 |
| `requirements.md` 5 ／ `handout_app_spec.md` 5-1 | 必須項目、文字数、空白、ジャンルの5種類の確認。 |
| `requirements.md` 6 ／ `handout_app_spec.md` 5-4 | `title`・`category` による検索と `due_date` による並び替え。 |
| `requirements.md` 7 ／ `handout_app_spec.md` 5-5 | 操作記録に使う `id` と、記録しない `title`・`detail`。 |
