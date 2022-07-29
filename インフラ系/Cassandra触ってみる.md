# what
cassandraを自分のローカル環境に入れて触ってみた時の個人的ログ

# 今回のゴール
ローカルにcassandraの環境を構築して以下のことをやってみる
- テーブルを作ってみる
- テーブルのデータを追加・変更・削除してみる

# やってみる

### 必要なもの
docker


## 環境構築

dockerhubに記載されているcassandraの環境構築を参考に作ってみる
ref: [cassandra dockerhub](https://hub.docker.com/_/cassandra)

今回用にdockerのネットワーク環境を作成する

```zsh
$ docker network create cassandra-nw
```

ローカルにデータを保存するディレクトリを作成
(今回は2つのノードを作成するので、2つのディレクトリを用意)

```zsh
$ mkdir practice-cassandra
$ mkdir practice-cassandra-2
```

以下のコマンドを実行して、1つ目のcassandraのノードを作成する

```zsh
$ docker run --name cassandra1 --network cassandra-nw -v $HOME/practice-cassandra:/var/lib/cassandra -p 9042:9042 cassandra:latest
```

2つ目のノードも同様に作成

```zsh
$ docker run --name cassandra2 --network cassandra-nw -e CASSANDRA_SEEDS=cassandra1 -v $HOME/practice-cassandra-2:/var/lib/cassandra -p 9043:9042  cassandra:latest
```

### SEEDS(シード)とは

2つ目のノードを作成する際に`CASSANDRA_SEEDS`という環境変数を用いて、シードとなるノードを指定している。
シードノード…新しいノードをcassandraのリングに加える際に、ノードのゴシッププロセス(ノードが自身や他のノードの状態に関する情報を定期的に交換するピアツーピア通信プロトコル)を自動実行するために利用される。

ref:
 [ノード間のコミュニケーション(ゴシップ)](https://docs.datastax.com/ja/dse/5.1/dse-arch/datastax_enterprise/dbArch/archGossipAbout.html)
[単一データ・センターのシード・ノードの設定](https://docs.datastax.com/ja/dse/6.7/dse-admin/datastax_enterprise/production/seedNodesForSingleDC.html)



## CQLでクエリを叩く
cqlを起動するために以下コマンドを実行

```zsh
docker run -it --network cassandra-nw --rm cassandra cqlsh cassandra1
```

`cqlsh>` と表示されていれば、cqlでクエリを実行する環境が作成できている


## keyspaceを作る
※`keyspace`...mysqlでいうデータベースに相当する概念

まずはkeyspace一覧を表示
ref: [DESCRIBE KEYSPACE](https://docs.datastax.com/ja/dse/5.1/cql/cql/cql_reference/cqlsh_commands/cqlshDescribeKeyspace.html)

```zsh
DESCRIBE keyspace

system_schema  system_auth  system  system_distributed  test  system_traces

```

次に今回使用するkeyspaceを作成する(今回は`test`という名前で作成)

```zsh

cqlsh> CREATE TABLE test.feed (
   ...     user_id  text,
   ...     timestamp    timestamp,
   ...     message text,
   ...     primary key((user_id), timestamp)
   ... ) WITH CLUSTERING ORDER BY (timestamp DESC);
cqlsh> DESC test.food
```

## tableを作る
cassandraでは、テーブルをどのように作るかによって投げられるクエリが殆ど決まってしまうので、テーブル作成が大変重要(らしい)

```zsh
CREATE TABLE test.feed (
   ...     user_id  text,
   ...     timestamp    timestamp,
   ...     message text,
   ...     primary key((user_id), timestamp)
   ... ) WITH CLUSTERING ORDER BY (timestamp DESC);

# 作ったテーブルの確認

cqlsh> DESC test.feed

CREATE TABLE test.feed (
    user_id text,
    timestamp timestamp,
    message text,
    PRIMARY KEY (user_id, timestamp)
) WITH CLUSTERING ORDER BY (timestamp DESC)
    AND bloom_filter_fp_chance = 0.01
    AND caching = {'keys': 'ALL', 'rows_per_partition': 'NONE'}
    AND comment = ''
    AND compaction = {'class': 'org.apache.cassandra.db.compaction.SizeTieredCompactionStrategy', 'max_threshold': '32', 'min_threshold': '4'}
    AND compression = {'chunk_length_in_kb': '64', 'class': 'org.apache.cassandra.io.compress.LZ4Compressor'}
    AND crc_check_chance = 1.0
    AND dclocal_read_repair_chance = 0.1
    AND default_time_to_live = 0
    AND gc_grace_seconds = 864000
    AND max_index_interval = 2048
    AND memtable_flush_period_in_ms = 0
    AND min_index_interval = 128
    AND read_repair_chance = 0.0
    AND speculative_retry = '99PERCENTILE';
```

テーブル確認したら色々出てきたので順を追って見ていく

### Cassandraのデータタイプ
色々あるので↓を参照
[CQL data types](https://docs.datastax.com/en/cql-oss/3.3/cql/cql_reference/cql_data_types_c.html?hl=cql%2Cdata%2Ctypes)

### primary  key とclustering keyについて

cassandraでは`map<partitionkey`, `sortedmap<columnkey, value`という形でデータを保持している。
今回作成したfeedテーブルではprimary keyとして`((user_id), timestamp)`、clustering keyとしてtimastamp DESCを指定している。
これによりclustering keyに指定されていないuser_idはpartition keyとなる。
partition key単位でノードにレコードが配置される。

clustering keyとして指定されたカラムはカラム名ではなく、カラムの値がColumnKeyとなる。
そして，clustring keyの値は，他のprimary keyでないカラム名のプレフィックスとなる。
今回の例ではclustring keyの値が`timestampの値: message`となる。


## データ追加と更新

```zsh
# データ追加
cqlsh> INSERT INTO test.feed(user_id, timestamp, message) VALUES ('letitbe_or_not', '2019-03-25 00:00:000', 'あ');
cqlsh> INSERT INTO test.feed(user_id, timestamp, message) VALUES ('letitbe_or_not', '2019-03-25 00:01:000', 'い');
cqlsh> INSERT INTO test.feed(user_id, timestamp, message) VALUES ('letitbe_or_not', '2019-03-25 00:02:000', 'う');
cqlsh> INSERT INTO test.feed(user_id, timestamp, message) VALUES ('twitter', '2019-03-25 00:00:000', 'Work, work, work, work, work, work');
cqlsh> INSERT INTO test.feed(user_id, timestamp, message) VALUES ('twitter', '2019-03-25 00:01:000', 'Hahahahahahahaha');
cqlsh> INSERT INTO test.feed(user_id, timestamp, message) VALUES ('twitter', '2019-03-25 00:02:000', 'Go to sleep');
cqlsh> SELECT * FROM test.feed
   ...
cqlsh> SELECT * FROM test.feed;

 user_id        | timestamp                       | message
----------------+---------------------------------+------------------------------------
        twitter | 2019-03-25 00:02:00.000000+0000 |                        Go to sleep
        twitter | 2019-03-25 00:01:00.000000+0000 |                   Hahahahahahahaha
        twitter | 2019-03-25 00:00:00.000000+0000 | Work, work, work, work, work, work
 letitbe_or_not | 2019-03-25 00:02:00.000000+0000 |                                 う
 letitbe_or_not | 2019-03-25 00:01:00.000000+0000 |                                 い
 letitbe_or_not | 2019-03-25 00:00:00.000000+0000 |                                 あ

(6 rows)

# データ更新
cqlsh> UPDATE test.feed SET message='え' WHERE user_id = 'letitbe_or_not' AND timestamp= '2019-03-25 00:01:000';
cqlsh> SELECT * FROM test.feed;

 user_id        | timestamp                       | message
----------------+---------------------------------+------------------------------------
        twitter | 2019-03-25 00:02:00.000000+0000 |                        Go to sleep
        twitter | 2019-03-25 00:01:00.000000+0000 |                   Hahahahahahahaha
        twitter | 2019-03-25 00:00:00.000000+0000 | Work, work, work, work, work, work
 letitbe_or_not | 2019-03-25 00:02:00.000000+0000 |                                 う
 letitbe_or_not | 2019-03-25 00:01:00.000000+0000 |                                 え
 letitbe_or_not | 2019-03-25 00:00:00.000000+0000 |                                 あ
```


## データの削除
以下のような操作でデータを削除する事が可能
- partitionKeyとClusteringKeyを指定して削除する。
- partitionKeyを指定して、特定Partitionのデータをすべて削除する

更に、partitioneKeyとClusteringKeyを統合で指定して、IF句を用いることで、primarykey出ないカラムに対しても条件を指定して削除することができる。

```zsh
DELETE FROM test.feed WHERE user_id = 'letitbe_or_not' AND timestamp = '2019-03-25 00:00:00';

cqlsh> SELECT * FROM test.feed;

 user_id        | timestamp                       | message
----------------+---------------------------------+------------------------------------
        twitter | 2019-03-25 00:02:00.000000+0000 |                        Go to sleep
        twitter | 2019-03-25 00:01:00.000000+0000 |                   Hahahahahahahaha
        twitter | 2019-03-25 00:00:00.000000+0000 | Work, work, work, work, work, work
 letitbe_or_not | 2019-03-25 00:02:00.000000+0000 |                                 う
 letitbe_or_not | 2019-03-25 00:01:00.000000+0000 |                                 え
```

# 参考資料
[Cassandraに入門した](https://getumen.github.io/2019/03/25/cassandra%E3%81%AB%E5%85%A5%E9%96%80%E3%81%97%E3%81%9F/)
