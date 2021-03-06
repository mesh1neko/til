# 実際に負荷をかけてみた
＠Node.js(Express) on Docker

DockerのNode.js環境ができたので、
JMeterで負荷をかけて、性能の計測をしてみたいと思います。

## 構成
* macをホストOS 兼 負荷クライアント
* 負荷サーバはゲストOS内のDockerコンテナ

// 最終的にはコンテナ to コンテナにしたい

イメージ図
```
// 負荷クライアント
[host:OS X(mac) with JMeter]
    |
    v                    // 負荷サーバ
[guest:boot2docker- - - >[Node.js(express):container]]
```

# スペック/バージョン情報
```
# スペック
$ sw_vers
ProductName:	Mac OS X
ProductVersion:	10.11.3
BuildVersion:	15D21
$ system_profiler SPHardwareDataType | head -n 14
Hardware:

    Hardware Overview:

      Model Name: MacBook Pro
      Model Identifier: MacBookPro11,1
      Processor Name: Intel Core i5
      Processor Speed: 2.4 GHz
      Number of Processors: 1
      Total Number of Cores: 2
      L2 Cache (per Core): 256 KB
      L3 Cache: 3 MB
      Memory: 8 GB
      Boot ROM Version: MBP111.0138.B16

# dcoker&Node
$ docker-machine --version
docker-machine version 0.7.0, build a650a40
$ docker --version
Docker version 1.11.1, build 5604cbe

$ docker exec -it 7de /bin/bash
---@container
# node --version
v4.4.5
# npm --version
2.15.5
---
```

# 性能試験のためのテストシナリオ作成
## 試験設定
以下のように設定し、stress-test.jmxとして保存.
* /へアクセスのみ
* スレッド数：1
* RampUp期間：10
* ループ回数：100,000

1 * 100,000なので単純計算で10秒で100,000リクエスト送る.

# 性能試験の実施
```
$ ../apache-jmeter-3.0/bin/jmeter -n  -t ./stress-test.jmx -l test.jtl
Writing log file to: /Users/iida-ryota/Documents/til/jmeter/trial/jmeter.log
Creating summariser <summary>
Created the tree successfully using ./stress-test.jmx
Starting the test @ Sun Jun 05 16:21:55 JST 2016 (1465111315762)
Waiting for possible Shutdown/StopTestNow/Heapdump message on port 4445
summary +   3752 in 00:00:04 =  904.8/s Avg:     0 Min:     0 Max:    81 Err:     0 (0.00%) Active: 1 Started: 1 Finished: 0
summary +  64579 in 00:00:30 = 2152.6/s Avg:     0 Min:     0 Max:   151 Err:     0 (0.00%) Active: 1 Started: 1 Finished: 0
summary =  68331 in 00:00:34 = 2001.1/s Avg:     0 Min:     0 Max:   151 Err:     0 (0.00%)
summary +  31669 in 00:00:13 = 2350.4/s Avg:     0 Min:     0 Max:    16 Err:     0 (0.00%) Active: 0 Started: 1 Finished: 1
summary = 100000 in 00:00:48 = 2099.9/s Avg:     0 Min:     0 Max:   151 Err:     0 (0.00%)
Tidying up ...    @ Sun Jun 05 16:22:43 JST 2016 (1465111363475)
... end of run
# 体感結構時間かかりましたね。
```

## コマンドラインの結果をGUIで確認
以下でlogに出力したファイルを参照するとOK
[統計レポート]
 ->[全てのデータをファイルに出力]
  ->[参照]

※ただし、-lで出力されるJTLファイルはサンプルのそれぞれの結果なので試験内容によって重くなる
```
$ wc -l test.jtl
  100200 test.jtl
# ん...？+200?
$ du -h test.jtl
7.8M	test.jtl
```

// 読み込ませた時にエラーが出たのはこのせいか？

コマンドラインで実行結果のレポートを出力するにはコマンドライン用のプラグインがいるそうな

## レポート結果をGUIから出力してみた（csvファイル）
```
$ column -t -s, aggregate.csv
Label   # Samples  Average  Median  90% Line  95% Line  99% Line  Min  Max  Error %  Throughput  KB/sec
sample  100000     0        0       1         1         1         0    151  0.00%    2107.0      438.3
合計    100000     0        0       1         1         1         0    151  0.00%    2107.0      438.3
# こっちはちゃんと10万リクエスト扱い...なぞや！
$ grep '200,OK' -c test.jtl
100198
# リクエストのログには200,OKが100198あるらしい。なんでやねん。あとで調べてみよう。
```

# おまけ：レポートまでコマンドラインで済ませる
プラグインを入れればできるらしいので、やってみます。

## コマンドラインプラグインの導入
```
$ pwd
/Users/iida-ryota/Documents/til/jmeter/apache-jmeter-3.0
# jmeterのディレクトリで...
$ wget http://jmeter-plugins.org/downloads/file/JMeterPlugins-Standard-1.4.0.zip
--2016-06-05 20:42:18--  http://jmeter-plugins.org/downloads/file/JMeterPlugins-Standard-1.4.0.zip
Resolving jmeter-plugins.org... 192.169.82.254
Connecting to jmeter-plugins.org|192.169.82.254|:80... connected.
HTTP request sent, awaiting response... 302 Found
Location: /files/JMeterPlugins-Standard-1.4.0.zip [following]
--2016-06-05 20:42:19--  http://jmeter-plugins.org/files/JMeterPlugins-Standard-1.4.0.zip
Reusing existing connection to jmeter-plugins.org:80.
HTTP request sent, awaiting response... 200 OK
Length: 1266201 (1.2M) [application/zip]
Saving to: 'JMeterPlugins-Standard-1.4.0.zip'

JMeterPlugins-Standard-1.4.0.zip   100%[==================================================================>]   1.21M   349KB/s   in 3.5s   

2016-06-05 20:42:23 (349 KB/s) - 'JMeterPlugins-Standard-1.4.0.zip' saved [1266201/1266201]
# 展開
$ unzip JMeterPlugins-Standard-1.4.0.zip
Archive:  JMeterPlugins-Standard-1.4.0.zip
  inflating: lib/ext/JMeterPluginsCMD.sh  
  inflating: lib/ext/TestPlanCheck.sh  
  inflating: lib/ext/FilterResults.bat  
  inflating: lib/ext/TestPlanCheck.bat  
  inflating: lib/ext/JMeterPluginsCMD.bat  
  inflating: lib/ext/FilterResults.sh  
  inflating: lib/ext/JMeterPlugins-Standard.jar  
  inflating: lib/ext/CMDRunner.jar   
# とりあえず上書きしない
replace LICENSE? [y]es, [n]o, [A]ll, [N]one, [r]ename: n
replace README? [y]es, [n]o, [A]ll, [N]one, [r]ename: n

# さきほどの実行ログからレポートを生成してみる
$ ./lib/ext/JMeterPluginsCMD.sh --generate-csv report.csv --input-jtl ../trial/test.jtl --plugin-type AggregateReport

# WARNがいくつかコンソールに流れたけど、無事にできたようです。
$ column -s, -t < report.csv | cat
sampler_label  aggregate_report_count  average  aggregate_report_median  aggregate_report_90%_line  aggregate_report_min  aggregate_report_max  aggregate_report_error%  aggregate_report_rate  aggregate_report_bandwidth  aggregate_report_stddev
sample         100000                  0        0                        1                          0                     151                   0.00%                    2051.3                 426.7                       1.22
合計           100000                  0        0                        1                          0                     151                   0.00%                    2051.3                 426.7                       1.22

# 値がさっきと微妙に違うのはテストをやり直して、結果が違うだけね！

```
# 参考
[JMeterをコマンドラインで実行＆レポート作成](http://christina04.hatenablog.com/entry/2015/04/07/213110)
