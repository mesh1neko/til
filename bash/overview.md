# bash全体のtips

## built inコマンド
bashにもともと組み込まれているコマンドこと。
ここで対比がわかりやすいように内部・外部と呼び分ける。

普段使っているコマンドにも実は種類があったのだ。
* 内部コマンド：もともとbashの組み込みコマンド
* 外部コマンド：プログラムの実体があるコマンド

内部コマンドか外部コマンドかはtype -aで確認できる。
```
$ type -a cd
cd is a shell builtin # 内部
cd is /usr/bin/cd # 外部

$ type -a echo
echo is a shell builtin # 内部
echo is /bin/echo　# 外部

$ type -a source
source is a shell builtin　# 内部しか無い

$ type -a git
git is /usr/bin/git # 外部しかない
```

cf. [Linux上でシェルが実行される仕組みを，体系的に理解しよう　（bash 中級者への道）](http://language-and-engineering.hatenablog.jp/entry/20110617/p1)

### 同名のコマンドが存在する時
> コマンドをパス名なしで実行すると、シェルは以下の順に、該当するコマンドを探すそうです。
* 「（1）エイリアス」
* 「（2）ビルトインコマンド（内部コマンド）」
* 「（3）PATHに指定されたディレクトリ」

## quoting
> 特定の文字や単語が持つシェルに対する特別な意味をなくせます。変数展開も防止。
つまり、エスケープと関連が深い。

> $'string' の形式を持つ単語は特殊な扱いを受けます。
展開された結果はシングルクォートされているのと同じ状態で、 ドル記号は存在しなかったかのように扱われます。

e.g.
* $'¥n':改行文字
* $'¥t':タブ
* $'¥'':シングルクォート
* $'¥"':ダブルクォート

## オプションの終わり --
> -- はオプションの終わりを示し、それ以降のオプション処理を行いません。
> -- 以降の引き数は全て、ファイル名や引き数として扱われます。 引き数 - は -- と同じです。
渡す引数がオプション名とかぶったりすることもあるでしょう、うん。

## 算術式展開:$(())
bashで演算をするときにお世話になる。exprよりオススメ。

> Arithmetic Expansion
>       Arithmetic  expansion allows the  evaluation of an arithmetic expression
>       and the substitution of the result.  The format for  arithmetic  expansion is:

>     $((expression))
> from man of bash

```
$ echo 1+2 # そのまま書くと文字列扱い
1+2
$ echo $((1+2)) # 数式として処理する
3
$ echo $(( (1+2)/3 )) # わかりやすくスペースを空けてる
1
```

## 他のスクリプトを読み込む
.(dot)を指定するとできます。

```
### 呼び出されるファイルです
$ cat function.sh
#!/bin/bash
msg="hogehoge"

function showMsg() {
    echo "msg is "$1
}

### 呼び出すファイルです
$ cat main.sh
#!/bin/bash
echo "This is $0"

## この時点では定義されていません
echo $hoge
showMsg "msg"

. ./function.sh # ここで読み込まれます

## これ以降は読み込まれた内容が反映されるはずです
echo $msg
showMsg "msg"

### 実際に実行してみます
$ ./main.sh
This is ./main.sh
 # でてないですね
./main.sh: line 5: showMsg: command not found # not foundですね！
hogehoge # でてますね！
msg is msg # こっちもでてますね！

```

## スクリプトで環境変数の設定
普通にやっても、実行プロセスで環境が閉じているため、反映されません
```
$ export -p | grep -c TEST_ENV_VARIABLE
0
$ ./script.sh
$ export -p | grep -c TEST_ENV_VARIABLE
0
```

実行結果をsourceコマンド読み込みます
```
$ export -p | grep -c TEST_ENV_VARIABLE
0
$ source ./script.sh
$ export -p | grep -c TEST_ENV_VARIABLE
1
```
