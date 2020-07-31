---
title: Sqlite3 の排他制御のテスト
---


## 概要

SQLite の排他制御の動作が不思議だったので、検証コードを作ってテストした。

* 片方が transation 中に、もう片方がアクセスすると 
  SQLite3::BusyException になる。( その時は retry はしていなかった。)

* busy_timeout(5000) を指定しても効いて無いように見える。

## 検証コード

```ruby
require 'rubygems'
require 'sqlite3'

DBfile = "./test.db"

unless File.exist?( DBfile )
  db = SQLite3::Database.new(DBfile )
  sql = <<EOS
create table  data (
    id        integer  primary key,
    name      text
);
EOS
  db.execute_batch(sql)
  db.close()
end

0.upto( 100 ) do |n|
  db = SQLite3::Database.new(DBfile )
  ecount = 0
  begin
    db.busy_timeout(5000)
    db.transaction do
      STDERR.print(".")
      STDERR.flush()
      db.execute("insert into data ( name ) values ( ? )", n )
      rand(11).downto( 0 ) do |n|
        STDERR.print("-")
        STDERR.flush()
        sleep(1)
      end
    end
  rescue SQLite3::BusyException
    STDERR.print("*")
    STDERR.flush()
    if ecount > 10
      print ">SQLite3::BusyException\n"
      exit
    end
    ecount += 1
    sleep(1)
    retry
  rescue => e
    p $!
    exit
  end
  db.close()
  STDERR.print("\n")
  STDERR.flush()
  sleep( 1 )
end
```


## 実験

コンソールを 2つ開き、同じディレクトリで上記のプログラムを実行する。

+ コンソール1
```
.-----
.*.----------
.*.*.-
.*.*.----
.*.-----------
.*.*.-
.*.-----------
.*.--------
.*.*.--
```



+ コンソール2
```
.*.*.-------
.*.-----
.*.*.-
.*.-
.*.-
.*.*.-----
.*.----------
```


## 判った事

+ 何故か transaction が競合すると、
  1回目は busy_timeout(5000) は効かず例外が起きる。
  その後 rerty すると有効になる。
  <br>
  ただし FreeBSD の SQLite3.7.3 は一発目からちゃんと待つ。
  待たないのは Linux の SQLite 3.3.6 

+ なので、いずれにせよちゃんと retry してやれば、transaction        
  でもアクセスの競合を防ぐ排他制御が可能
