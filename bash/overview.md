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