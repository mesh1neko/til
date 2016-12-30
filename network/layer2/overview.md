# OSI参照モデルのL2の話
L2ではL3までのパケットに制御情報を付加したフレームというデータユニットの伝送制御を行う。

## L2の副層
実はL2にはより詳細に役割を分けたLLC副層とMAC副層の２種類の取り決めが存在する。
※OSIの7層はあくまでモデルなので、それぞれ事細かにあってもおかしくない。

### LLC（ Logical Link Control Sub-Layer）：論理リンク制御副層
機器に依存しない、エラー制御などの部分を取り決めている。
しっかりとしたエラー制御はL4で行うため、ここではビットチェック程度のチェックを実施するそうだ。

### MAC（Media Access Control Sub-Layer）：メディアアクセス制御副層
メディアアクセスと書くと難しく聞こえるが、誰が送信するか？についての取り決め。CSMA/CD方式と関係あり？

具体的には制御情報に含まるMACアドレスの付与や削除をおこなっている模様。

## LANの規格から見るL2層
LANに用いられる規格には次の5つがあげられる。

* イーサネット
 * L1~L2
* IEEE802.2
 * L2-LLC
* IEEE802.3
 * L1~L2-MAC
* IEEE802.5
 * L1~L2-MAC
* FDDI
 * L1~L2-MAC

ただし、IEEE802.2に関しては機器に依存しない規格のため、
物理的なLANの規格で考慮すると、それぞれがIEEE802.2を含んでいると考えて
イーサネット, IEEE802.3, IEEE802.5, FDDIがそれに該当するようです。

なお、イーサネットとIEEE802.3はほぼ同規格らしいです。

## L2におけるアドレッシング
デバイスを識別するための記号、アドレッシングについて。識別するためにはユニークである必要があります。

アドレスには論理アドレスと物理アドレスがありますが、L2では物理アドレスにスポットを当てていきます。

### 論理アドレス
最終的に届ける場所。荷物の配送でいうなら宛先にあたる。

### 物理アドレス
次に届ける場所。荷物でいうとポストとか集配所、郵便局とか。トラックの荷台とか。

### MACアドレス
物理アドレスの１つ。NICにつけられた識別子で48bit、つまり16進12桁の文字列からなる。

ただし、表記では２桁ずつ-(ハイフン)で区切るようです。

e.g. 00-90-99-32-BA-FF

なお、8桁目まではベンダコードと呼ばれIEEEが決定しており、各ベンダは残り24bit（4桁）で割り振りをおこなっている。