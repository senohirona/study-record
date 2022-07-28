※SQLを書き始めた頃に書いたもの(2018年)


# what
- mysql のEXPLAINについて概念を記載する
- 実際に利用できるようにする

# why
- 処理が重いクエリを作成しても、改善できるようにするため

# EXPLAINとは

- クエリのパフォーマンスを確認する
    - [MySQLのEXPLAIN](https://qiita.com/mucho0623/items/fa575002bf0ade699e2f)
    - [MySQLのEXPLAINを徹底解説!! (少し古い記事)](http://nippondanji.blogspot.com/2009/03/mysqlexplain.html)
- 主にINDEXを貼るために利用されている
    - [MySQLでインデックスを貼る時に読みたいページまとめ(初心者向け）](https://qiita.com/C058/items/1c9c57f634ebf54d99bb)
    - INDEXについてざっくり書かれている。

## 利用法
- SELECT文の先頭に`EXPLAIN`をつけて実行する

例)
```sql
EXPLAIN SELECT カラム名1, カラム名2... FROM テーブル名 WHERE 条件...;
```

## 各フィールドの見方

### id/select
- idは実行の順番
- クエリの種類
- selectはクエリの種類
    - JOIN、サブクエリ、UNIONおよびそれらの組み合わせ
    - JSON
        - MySQLが実行できるJOINの種類は「Nested Look Join(NLJ)」の一種類しかない。
        - テーブルを一つずつ順に処理していく方式。
        - クエリがJOINだけから構成される場合、select_typeは「SIMPLE」と表示される。
SIMPLEではidが全て同じ値になる。
    - サブクエリ
        - PRIMARY・・・外部クエリを示す
        - SUBQUERY・・・相関関係のないサブクエリ
        - DEPENDENT SUBQUERY・・・相関関係のあるサブクエリ
        - UNCACHEABLE SUBQUERY・・・実行する度に結果が変わる可能性のあるサブクエリ
        - DERIVED・・・FROM句で用いられているサブクエリ
    - DERIVEDの場合: サブクエリ->外部クエリの順番でクエリが実行
    - それ以外の場合: 外部クエリ->サブクエリの順番でクエリが実行
    - サブクエリの場合、外部クエリとサブクエリでは別々のidがつけられる

### table
- アクセスする対象のテーブル

### type
- 対象のテーブルに対してどのような方法でアクセスするかを示す(レコードアクセスタイプというらしい)
- **致命的なクエリはここを見るとわかる**

#### typeの詳細
- const
    - PRIMARY KEYまたはUNIQUEインデックスのルックアップによるアクセス
    - 一番早い
- eq_ref
    - JOINにおいてPRIARY KEYまたはUNIQUE KEYが利用される時のアクセスタイプ
    - constのJSONが利用されているバージョン?
- ref
    - ユニーク（PRIMARY or UNIQUE）ではないインデックスを使って等価検索を行った時に使用されるアクセス
- range
    - INDEXを用いた範囲検索
- index
    - フルインデックススキャン
    - INDEX全体をスキャンするのでとても遅い
- ALL
    -  フルテーブルスキャン
    - INDEXが全く利用されていないことを示す
    - 一番重く、改善が必須となる状態

**<とりあえず>**
**indexまたはALLを見かけたらすかさずクエリをチューニングする**

### possible_keys
- オプティマイザがテーブルのアクセスに利用可能だと判断したINDEX

### key
- 実際にオプティマイザによって使用されたキー


### key_len
- 選択されたキーの長さ
- 短いほうが高速


### ref
- 検索条件で、keyと比較されている値やカラムの種類
    - 定数の場合
        - const
    - JSONを利用している場合
        - 結合する相手側のテーブルで検索条件として利用されているカラムが表示される

### row
- そのテーブルからフェッチされる行数の見積もり
- あくまでも「見積もり」なので正式な数ではないことに注意


### Extra
- そのクエリを実行するためにオプティマイザがどのような選択をしたかを表す
- Using where
    - 頻繁に出力される追加情報。
    - WHERE句に検索条件が指定されており、なおかつインデックスを見ただけではWHERE句の条件を全て適用することが出来ない場合に表示される。
- Using index
    - クエリがインデックスだけを用いて解決できることを示す。
    - Covering Indexを利用している場合などに表示される。
- Using filesort
    - filesort（クイックソート）でソートを行っていることを示す。
- Using temporary
    - JOINの結果をソートしたり、DISTINCTによる重複の排除を行う場合など、クエリの実行にテンポラリテーブルが必要なことを示す。
- Using index for group-by
    - MIN()/MAX()がGROUP BY句と併用されているとき、クエリがインデックスだけを用いて解決できることを示す。
- Range checked for each record (index map: N)
    - JOINにおいてrangeまたはindex_mergeが利用される場合に表示される
- Not exists
    - LEFT JOINにおいて、左側のテーブルからフェッチされた行にマッチする行が右側のテーブルに存在しない場合、右側のテーブルはNULLとなるが、右側のテーブルがNOT NULLとして定義されたフィールドでJOINされている場合にはマッチしない行を探せば良い・・・ということを示す。
