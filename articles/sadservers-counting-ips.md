---
title: "[SadServers] #2: 最もアクセスの多いIPを割り出す"
emoji: "😢"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["sadservers", "linux", "bash", "awk"]
published: true
---

# はじめに
この記事は SadServers の「"Saskatoon": counting IPs.」の解説になります。
https://sadservers.com/newserver/saint-john

# 状況
Web サーバーがあり、そこに対するアクセスを記録したログファイル: `/home/admin/access.log` があるようです。試しに `cat` コマンドで中身を見てみます。

```sh
$ cat /home/admin/access.log
66.249.73.135 - - [20/May/2015:21:05:00 +0000] "GET /?flav=atom HTTP/1.1" 200 32352 "-" "Mozilla/5.0 (compatible; Googlebot/2.1; +http://www.google.com/bot.html)"
180.76.6.56 - - [20/May/2015:21:05:56 +0000] "GET /robots.txt HTTP/1.1" 200 - "-" "Mozilla/5.0 (Windows NT 5.1; rv:6.0.2) Gecko/20100101 Firefox/6.0.2"
46.105.14.53 - - [20/May/2015:21:05:15 +0000] "GET /blog/tags/puppet?flav=rss20 HTTP/1.1" 200 14872 "-" "UniversalFeedParser/4.2-pre-314-svn +http://feedparser.org/"

...

```

どうやら、一行ごとに HTTP リクエストの内容が記録されているようです。
行の先頭はIPアドレスであり、それに続けていくつかの情報が記録されていることが分かります。

# 課題
このログのうち、 **もっともアクセス回数の多いIPアドレス** を割り出したいとします。問題によると、`/home/admin/highestip.txt` にIPアドレスを記載するのがゴール条件のようです。

# 解決法
さて、解法に移ります。

まず、不要な情報を取り除いて、必要な情報だけを抜き出す必要がありますね。今回の場合、必要なのはIPアドレスであり、それ以外のものは不要だといえそうです。IPアドレスは行の先頭にあるので、一行ずつスペース区切りで行の先頭にある文字列を抜き出す方法が分かればよさそうです。

Linux で一行ずつ処理を行う必要がある場合、 **`awk` （オーク）** が役立ちます。これはコマンドであり、ひとつのプログラミング言語です。そのため、他の言語と同じように Hello World ができます。

```sh
$ awk '{ print "Hello World! }'
Hello World!
```

さて、`awk` は他のコマンドの結果を入力として受けとり、処理をすることができます。こうしたコマンドを **フィルタコマンド** と呼びます。`awk` に複数行のテキストを渡した場合、書かれた処理を行ごとに実行してくれます。
そして、`awk` はスペース区切りの文字列を **フィールド** という一時的な変数に分割して代入してくれます。スペース区切りで2つめの単語は、`$2` といった具合です。

```sh
$ cat sample.txt
hoge huga piyoyo

$ cat sample.txt | awk 'print $1'
hoge
```

これらの知識を組み合わせて、IPだけを抜き出すことができそうです。IPはスペース区切りで1番目に記述されているので、`$1` を `print` してあげます。

```sh
$ cat /home/admin/access.log | awk '{print $1}'
66.249.73.135
180.76.6.56
46.105.14.53

...

```

見事、IPだけを抜き出すことができました。

さて、ここからは計測のターンです。とはいっても、手作業で数えるのは現実的ではありません（SadServers の制限時間は 15 分しかないので）。そこで、`sort` と `uniq` という2つのコマンドを使用して、重複する要素のカウントに挑戦してみましょう。

まずは `uniq` から説明します。これは「unique」の略で、反復する行（「重複する行」ではありません）を削除したり、カウントしたりするのに使います。`uniq` だけだと反復する行は消えてしまいますが、`uniq -c` のようにオプションを付けてあげることで、反復する行をカウントしてくれます。

```sh
$ cat sample.txt
hoge
hoge
hoge

$ cat sample.txt | uniq -c
      3 hoge
```

しかし、単に `uniq` を使用すればカウントしてくれるわけではありません。先ほど「重複する行ではない」と注釈を入れたのには訳があります。それは、「連続する重複行しかカウントしてくれない」からです。

```sh
$ cat sample.txt
hoge
huga
hoge

$ cat sample.txt | uniq -c
      1 hoge
      1 huga
      1 hoge
```

そこで `sort` の出番です。`sort` は文字列を並び替えてくれますが、副次作用として「同じ文字列は並べて配置してくれる」といった特性があります。これを利用して、`uniq` がカウントをしやすいようにファイルを整理することができます。

```sh
$ cat sample.txt
hoge
huga
hoge

$ cat sample.txt | sort | uniq -c
      2 hoge
      1 huga
```

さて、これを問題に適用する時です。先ほどまでの `awk` コマンドの結果を受け取り、`sort` で同じIPを並べ...

```sh
$ cat /home/admin/access.log | awk '{print $1}' | sort
1.22.35.226
1.22.35.226
1.22.35.226

...

100.2.4.116
100.2.4.116
100.2.4.116

...

```

その結果を `uniq` でカウントします。

```sh
$ cat /home/admin/access.log | awk '{print $1}' | sort | uniq -c 
      6 1.22.35.226
      6 100.2.4.116
     84 100.43.83.137

...

```

いいですね！それぞれの IP について、アクセス回数がカウントできました。

最後に、もう一度昇順にソートします。最も大きいIPは最後の行に表示されるはずなので、どのIPがナンバーワン化を知ることができます。
ただし、ひとつ注意事項があります。`sort` はそのまま用いると **文字列を** ソートします。今回は最も大きい **数字** を知りたいので、このままでは適しませんね。
`sort` で数字を比較するには、`-n` オプションを付けるだけです。

```sh
$ cat /home/admin/access.log | awk '{print $1}' | sort | uniq -c | sort -n
...
    113 50.16.19.13
    273 75.97.9.59
    357 130.237.218.86
    364 46.105.14.53
    482 66.249.73.135
```

最も大きいアクセスは `66.249.73.135` で、その回数は 482 回であることが分かりましたね。

# 参照
https://sadservers.com/newserver/saint-john