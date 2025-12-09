---
title: "マイグレーションの事例で考える別技術の習得"
emoji: "🔁"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["マイグレーション", "Rails", "Prisma", "Go"]
published: true
---

この記事は [デイトラプログラミングコース Advent Calendar 2025](https://adventar.org/calendars/11304) 10日目の記事です。

皆さんこんにちは！
デイトラWebアプリ開発コースでメンターを担当している小林です！

ほとんどのエンジニアは現場に出る、もしくは新言語のキャッチアップを通して、新しい言語に触れる機会が必ずやってきます。
そんな時、「やりたいことがあるのに、知ってる言語とやり方が全然違う！どうしよう……」と不安になったことはありませんか？

実は、新しい技術を学ぶ際に「ゼロから覚える」必要はありません。
すでに皆さんが持っている言語の知識を基準にして、**「何が同じで、何が違うのか」を整理することで、スムーズにキャッチアップできるようになります。**

今回は、Web開発に欠かせない「マイグレーション」を題材に、複数のツール（Railsのmigrate, Ridgepole, Prisma, golang-migrate）を比較しながら、私が新しい技術を学ぶ時の考え方をまとめていきます。
なお、これら全ての技術を過去や現在の案件で使用したり、ゼロから整備したりしてきたので、本当の事例となります。

## Railsのマイグレーションのおさらい
まず、比較の基準となるRailsのマイグレーションを整理しましょう。(Rails以外の方はこのようにマイグレーションするんだなと思ってください)
Railsのマイグレーションでは、「データベースに対してどのような変更操作を行うか」をバージョン管理して記述するスタイルを取っています。

【実行フロー】
- `$ rails g model User`を実行し、マイグレーションファイルを生成する。
- 生成されたマイグレーションファイルを編集して、カラムなどを追加する。
- `$ rails db:migrate`を実行して`db/schema.rb`を更新し、変更をデータベースに適用していく。

【コード例】

```ruby
# db/migrate/20251210123456_create_users.rb
class CreateUsers < ActiveRecord::Migration[7.0]
  def change
    create_table :users do |t|
      t.string :name
      t.timestamps
    end
  end
end
```

【ポイント】
- バージョン管理: 変更履歴（Create Table, Add Columnなど）をファイルとして積み重ねていく。
- Rubyを使用: Rubyのコードで書くため、SQLを直接意識しなくて良い。

## 他のツールは？
では、他のマイグレーションツールを見てみましょう。
Railsと比較することで特徴が浮き彫りになります。

### ケース1：Ridgepole (Rails)
Ridgepoleはクックパッド社発のツールで、gem(ライブラリ)の一つですが、概念はRails標準と大きく異なります。
このgemを使うと、SchemaファイルというファイルにRubyでテーブル定義を書いて、コマンドを実行するだけでデータベースに変更が適用されます。

【実行フロー】
- `Schemafile`という一つのファイルを編集し、テーブル定義を書く。
- `$ bundle exec ridgepole --apply`を実行する。
- ツールが自動的に「現在のDB」との差分を比較し、必要なSQLなどを発行して同期させる。

【コード例】

```ruby
create_table "users", force: :cascade do |t|
  t.string "name"
  t.datetime "created_at", null: false
  t.datetime "updated_at", null: false
end
```

【Railsと異なる点】
- ファイル管理: 一つのファイルだけで管理するため、カラムの追加や削除時にも一つのファイル更新で良い。
- 冪等性(べきとうせい): 何度コマンドを実行しても、定義ファイルとDBの状態が一致するようにツールが変更をDBに適用してくれる。

### ケース2：Prisma (Node.js/TypeScript)
Prismaは、Node.js、TypeScript向けのORMです。
Railsとはアプローチが逆となっています。

【実行フロー】
- `schema.prisma`という定義ファイルを直接編集し、モデルのあるべき姿を考えて定義を記述する。
- `$ npx prisma migrate dev`を実行する。
- Prismaが **現在のDBとschema.prismaの差分を検知し、** SQLファイルを自動生成・実行する。

【コード例】

```prisma
model User {
  id        Int      @id @default(autoincrement())
  name      String
  createdAt DateTime @default(now()) @map("created_at")
  updatedAt DateTime @default(now()) @updatedAt @map("updated_at")
}
```

【Railsと異なる点】
- アプローチの逆転: Railsでは「マイグレーションファイルを書く→スキーマが決まる」という順番ですが、Prismaは「スキーマを書く→マイグレーションファイルが作られる」という順序です。
- 直感的: 「どうやって変更するか」ではなく「DBが最終的にどうなっていて欲しいか」を書くため、DBの最終的な定義を考えればOKです。

### ケース3：golang-migrate (Go)
`golang-migrate`では、フレームワークに頼らないシンプルなSQLを使用します。
そのため、生のSQLで`CREATE TABLE`や`ALTER TABLE`を実行します。

【実行フロー】
- マイグレーション作成コマンドを実行し、up.sql（適用用）と down.sql（ロールバック用）の空ファイルを生成する。
- そのファイルに生のSQLを自分で記述する。
- `migrate`コマンドを実行したり、コードからマイグレーションを実行したりしてDBに適用する。

【コード例】

```sql
CREATE TABLE users (
  id INT PRIMARY KEY,
  name VARCHAR(255) NOT NULL,
  created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP
  updated_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```

【Railsと異なる点】
- 生SQL: Railsの`t.string`のような便利な書き方はせず、直接`VARCHAR`や`CREATE TABLE`などを記述します。
- 固有の機能: とあるデータベース固有の機能を使いたい場合、直接その機能を使用できます。

## 共通点
さて、これまで4つのマイグレーションツールを見てきました。
一見バラバラに見えるこれらのツールですが、エンジニアの視点で「抽象化」すると、**やっていることは非常によく似ている**ことに気づきます。

どのツールも、大きく分ければ以下の要素で成り立っています。
- 定義する: マイグレーションファイルやスキーマファイルといったファイルを作成する。
- 適用する: 最終的にSQLを発行し、データベースに変更を加える。
- 学ぶ: SQLを直接書かないツール（RailsやPrismaなど）の場合、それぞれのツール独自の文法でファイルを書く。

### 脳内変換する
では、これらのような新しい技術を学ぶとき、私は何を考えていたのでしょうか？
実は、以下のように脳内で変換を行っています。

1. 基準と紐付ける
- 「これはRailsで言うところの rails g model にあたる操作だな」
- 「Railsのマイグレーションファイルでは`t.string`で型指定していたけど、Prismaにおけるモデルでは`String`と書くだけなのか」
→ 既知の技術（Rails）を「辞書」として使うイメージです。

2. 差分を言語化する
- 「Railsではマイグレーションファイルを先に書くけど、PrismaやRidgepoleはスキーマファイルを書けばいいんだ。ここが違うな」
- 「golang-migrateはSQLを書かせるんだな。SQLなら書いたことがあるから書けるな。」
→ 「何が同じで、何が違うか」を明確にすることで感覚をつかみやすくなります。

## 最後に
ツールや言語が変わっても、「データベースの構造を安全に変更・管理したい」といった最終的な目的は変わりません。
皆さんはすでに、その目的を達成するための基本的な思考法と知識（Railsという基準）を持っている可能性があります。
なので、もし今後現場で見たことのない言語やライブラリに出会ったら、**「RailsやJavaのあの操作や書き方と比べてどう違うんだろう？」** と考えてみてください。
そうすればスクール時代よりもすんなりと新しい技術の習得が出来るはずです。

## おまけ
推しグループが本日新曲を出したので皆さん聞いて下さい(ここで言うな)
かっこいい曲です！！！

https://www.youtube.com/watch?v=aXp14lrdymc
