
pt-snap-statements
==================

概要
----

2つの時刻のSQL文の統計情報の差分を計算して表示します。

オプションを指定することにより、特定の項目ごとにソートして表示することができます。

contribモジュールの ``pg_stat_statements`` が導入されている必要があります。

また、 ``track_io_timing`` オプションが有効にされている必要があります。


実行方法
--------

.. code-block:: none

   pt-snap-statements [option...] [interval]

オプション
----------

.. code-block:: none

  -h, --host=HOSTNAME
  -p, --port=PORT
  -U, --username=USERNAME
  -d, --dbname=DBNAME
  -s, --sort=KEY
  -l
  -t, --top=NUMBER
  -R, --reset
  --help

``-h``, ``--host`` オプションは、接続するPostgreSQLデータベースのサーバ名またはIPアドレスを指定します。オプションが指定されない場合は、PGHOST環境変数に設定された値が使われます。PGHOST環境変数が設定されていない場合には、デフォルトの値として ``localhost`` が使われます。

``-p``, ``--port`` オプションは、接続するPostgreSQLデータベースのポート番号を指定します。オプションが指定されない場合は、PGPORT環境変数に設定された値が使われます。PGPORT環境変数が設定されていない場合には、デフォルトの値として ``5432`` が使われます。

``-U``, ``--username`` オプションは、PostgreSQLデータベースに接続するユーザ名を指定します。オプションが指定されない場合は、PGUSER環境変数に設定された値が使われます。PGUSER環境変数が設定されていない場合には、USER環境変数に設定された値が使われます。

``-d``, ``--dbname`` オプションは、接続するデータベース名を指定します。オプションが指定されない場合は、PGDATABASE環境変数に設定された値が使われます。PGDATABASE環境変数が設定されていない場合には、データベースに接続するユーザ名と同じ名前のデータベースに接続します。

``-s`` オプションは、ソートする項目を指定します（未実装）。``KEY`` は、次のいずれかの値を取ることができます: ``CALLS``, ``T_TIME``, ``ROWS``, ``B_HIT``, ``B_READ``, ``B_DIRT``, ``B_WRTN``, ``R_TIME``, ``W_TIME``

``-l`` オプションは、ブロックの種別（共有バッファ、ローカルバッファ、一時バッファ）ごとに詳細な内訳を表示します（未実装）。``-l`` オプションを指定しない場合は、共有バッファ、ローカルバッファ、一時バッファの数値を合算した値が表示されます。

``-t``, ``--top`` オプションは、表示するクエリの数を指定します。指定しない場合はすべてのクエリが表示されます。

``-R``, ``--reset`` オプションは、``pg_stat_statements`` ビューの統計情報を初期化します。


出力項目
--------

.. csv-table::

   ``USER``, クエリを実行したユーザ名
   ``DBNAME``, クエリを実行したデータベース名
   ``QUERYID``, 実行されたクエリのクエリID（16進数表記）
   ``QUERY``, 実行されたクエリ（最大30文字で切り詰め）
   ``CALLS``, クエリの実行回数
   ``T_TIME``, クエリの総実行時間（ミリ秒）
   ``ROWS``, クエリによって取得または影響を受けた行の総数
   ``B_HIT``, ブロック読み込みの際にバッファから読み込んだブロック総数
   ``B_READ``, ブロック読み込みの際にディスクから読み込んだブロック総数
   ``B_DIRT``, クエリによってページが更新されたページ総数
   ``B_WRTN``, クエリによってディスクに書き込まれたブロック総数
   ``R_TIME``, ディスクからのブロック読み込みにかかった総時間（ミリ秒） （``track_io_timing`` パラメータが有効になっている必要がある）
   ``W_TIME``, ディスクへのブロック書き込みにかかった総時間（ミリ秒） （``track_io_timing`` パラメータが有効になっている必要がある）


実行例
------

``postgres`` データベースに接続し、で5秒間に実行されたSQL文を総実行時間（``T_TIME``）の長い順にソートしてすべて表示します。

.. code-block:: none

   $ pt-snap-statements -d postgres 5
   +-------+----------+----------+--------------------------------+-------+--------+------+-------+--------+--------+--------+--------+--------+
   |  USER |  DBNAME  | QUERYID  |             QUERY              | CALLS | T_TIME | ROWS | B_HIT | B_READ | B_DIRT | B_WRTN | R_TIME | W_TIME |
   +-------+----------+----------+--------------------------------+-------+--------+------+-------+--------+--------+--------+--------+--------+
   | snaga | postgres | 80053daf | UPDATE pgbench_branches SET bb |   677 |  12007 |  677 |  9160 |      1 |      1 |      0 | 0.0    | 0.0    |
   | snaga | postgres | 1675159e | UPDATE pgbench_tellers SET tba |   681 |   7648 |  681 |  3403 |      0 |      0 |      0 | 0.0    | 0.0    |
   | snaga | postgres | ec088219 | UPDATE pgbench_accounts SET ab |   684 |    530 |  684 |  2289 |    585 |    568 |      0 | 125.9  | 0.0    |
   | snaga | postgres | 198383d  | SELECT abalance FROM pgbench_a |   682 |     73 |  682 |  2080 |      0 |      0 |      0 | 0.0    | 0.0    |
   | snaga | postgres | da8cc6f  | INSERT INTO pgbench_history (t |   676 |     34 |  676 |   704 |     12 |     10 |      0 | 0.0    | 0.0    |
   | snaga | postgres | d4e6bf94 | BEGIN;                         |   684 |      4 |    0 |     0 |      0 |      0 |      0 | 0.0    | 0.0    |
   | snaga | postgres | a81672e  | END;                           |   671 |      3 |    0 |     0 |      0 |      0 |      0 | 0.0    | 0.0    |
   | snaga | postgres | 8caa574  | select count(*) from pgbench_b |     1 |      0 |    1 |     4 |      0 |      0 |      0 | 0.0    | 0.0    |
   +-------+----------+----------+--------------------------------+-------+--------+------+-------+--------+--------+--------+--------+--------+
   $

ホスト ``192.168.1.101`` のポート ``5433`` で稼働しているPostgreSQLサーバの データベース ``postgres`` にユーザ ``snaga`` で接続し、5秒間に実行されたSQL文を総実行時間（``T_TIME``）の長い順にソートしてトップ5件を表示します。

.. code-block:: none

   $ pt-snap-statements --host 192.168.1.101 -p 5433 -U snaga -d postgres -t 5 5
   +-------+----------+----------+--------------------------------+-------+--------+------+-------+--------+--------+--------+--------+--------+
   |  USER |  DBNAME  | QUERYID  |             QUERY              | CALLS | T_TIME | ROWS | B_HIT | B_READ | B_DIRT | B_WRTN | R_TIME | W_TIME |
   +-------+----------+----------+--------------------------------+-------+--------+------+-------+--------+--------+--------+--------+--------+
   | snaga | postgres | 80053daf | UPDATE pgbench_branches SET bb |   503 |   9953 |  503 |  8430 |     14 |      7 |      0 | 0.6    | 0.0    |
   | snaga | postgres | 1675159e | UPDATE pgbench_tellers SET tba |   508 |   6483 |  508 |  2551 |     10 |      9 |      0 | 0.3    | 0.0    |
   | snaga | postgres | ec088219 | UPDATE pgbench_accounts SET ab |   511 |    560 |  511 |  1424 |    698 |    477 |      7 | 91.0   | 12.1   |
   | snaga | postgres | 198383d  | SELECT abalance FROM pgbench_a |   511 |     93 |  511 |  1550 |      0 |      0 |      0 | 0.0    | 0.0    |
   | snaga | postgres | da8cc6f  | INSERT INTO pgbench_history (t |   503 |     20 |  503 |   530 |     13 |     11 |      0 | 0.1    | 0.0    |
   +-------+----------+----------+--------------------------------+-------+--------+------+-------+--------+--------+--------+--------+--------+
   $

