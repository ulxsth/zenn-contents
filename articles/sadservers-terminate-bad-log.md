---
title: "[Linux] ファイルを使用しているプロセスを見つけ、止める"
emoji: "🛑"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["sadservers", "linux"]
published: true
---

# はじめに
この記事は SadServers の 1問目を解いた際のノートです。
SadServers は Web 上で Linux の操作を実践的に学べるサービスです。
https://sadservers.com/newserver/saint-john

# 状況
特定のファイルを専有し、書き込みを続けるシステムがあったとします。
例として、SadServers では「`/var/log/bad.log` にログを吐き続ける悪いプログラム」があります。

試しに、`/var/log/bad.log` が更新されているかを見てみます。`tail -f` コマンドは、ファイルに更新があった場合即座にファイルの最後の行を出力します。

コマンドを実行すると、毎秒ファイルが更新されていることが確認できます。
![](/images/sadservers-terminate-bad-log/2025-01-20-094655.gif)

このログを吐き出す諸悪の根源を突き止め、プロセスを停止させることが今回の目的です。

# 手順
## 書き込み中のプロセスを特定する
まずは、ファイルに対して書き込みを行っているプロセスを探します。
ファイルを開いているプログラムを探すには、`lsof` コマンドが適役です。

```sh
lsof <対象のファイルパス>
```

実際に実行してみると、諸悪の根源は`badlog.py`であることが明らかになります。
![alt text](/images/sadservers-terminate-bad-log/screenshot-lsof.png)

ちなみに、`fuser` コマンドでも同様に探せるようです。

```sh
fuser /var/log/bad.log
```

さて、問題のあるプログラムがなにかはっきりすることができました。
ここで重要なのは `PID` の項目。これが **プロセスを識別するためのID** になります。
実際にプロセスをストップさせるとき、この `PID` を指定することになります。

## プロセスを停止する
対象が明らかになったことで、プロセスを停止させることができるようになりました。
安全なプロセスの停止は、`kill` コマンドで実行できます（物騒ですね）。

このとき、さきほどメモした **PID** が役に立ちます。
```sh
kill <PID>
```

プロセスが終了しない場合は、強制的に終了することもできます。
```sh
kill -9 <PID>
```

## 停止したか確認する
さいごに、しっかりプロセスが止まったか確認してもよいでしょう。
もういちど `lsof` を実行し、対象のプロセスが表示されないことを確認してください。

```sh
lsof <対象のファイルパス>
```

SadServers 上でも確認します。
コマンドの結果が表示されないということは、このファイルに書き込みを行っているプロセスがないということです。
![alt text](/images/sadservers-terminate-bad-log/screenshot-check-lsof.png)

これで完了です。お疲れ様でした。

# 参照
https://sadservers.com/newserver/saint-john