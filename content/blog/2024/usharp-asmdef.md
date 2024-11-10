+++
title = "U# assembly definition が定義されていない旨のエラー対処"
date = 2024-10-01
[taxonomies]
tags = ["tech", "VRChat", "Unity", "UdonSharp", "memo"]
+++

公開用に `Packages/` 側で U# のアプリケーションを作っているときに、以下でビルドエラーになったのでの備忘録。

```log
'/Packages/XXX/XXX.cs' does not belong to a U# assembly, have you made a U# assembly definition for the assembly the script is a part of?
```

~~エラーメッセージの通りで書き残すほどでもない気はするが、U# Assembly Definition についてあまり情報がなかったので書き残す~~

_記事を書いている途中に公式ドキュメントを見つけました。 <https://udonsharp.docs.vrchat.com/migration/#does-not-belong-to-u-assembly>_

<!-- more -->

## Assembly Definition (asmdef) について

asmdef は定義済アセンブリを明示するための設定ファイル。

Unity C#で記述した Script がコンパイルされる際、 asmdef で定義を明示しない場合は `Assembly-CSharp*` にまとめられる。
自分の書いた Script とその他無関係の Script が同一のアセンブリにビルドされるため、クラス名の衝突であったりコンパイルの時間増加等の問題につながるため、特別な理由がなければ定義しておくほうが無難。

参考: [特殊フォルダーとスクリプトのコンパイル順 - Unity Documentation 2022.3](https://docs.unity3d.com/ja/2022.3/Manual/ScriptCompileOrderFolders.html)

## 今回のエラーについて

![error.png](/2024/usharp-asmdef/error.png)

今回の話は Unity C#ではなく、UdonSharp (U#) で発生した内容。同様に asmdef を配置していたが、掲題のエラーが発生した。
エラーメッセージにある通り、 U# の asmdef を作成しましたか？ というもの。asmdef だけでは U# 向けのアセンブリなのか判断できなかったと思われる。

asmdef は以下

![asmdef.png](/2024/usharp-asmdef/asmdef.png)

## 対処

U# Assembly Definition を追加する。右クリックで asmdef の下辺りにある。

![contextmenu.png](/2024/usharp-asmdef/contextmenu.png)

このファイルの設定には asmdef を指定する項目があるが、もともと定義してある asmdef を設定する。

![usharp-asmdef.png](/2024/usharp-asmdef/usharp-asmdef.png)

もし asmdef がなければ作成する。定義名や配置については公式ドキュメントに規則がある。U# Script そのものであれば Runtime 下になると思われる。

参考: [アセンブリ定義とパッケージ - Unity Documentation 2022.3](https://docs.unity3d.com/ja/2022.2/Manual/cus-asmdef.html)

これで無事にビルドできた。

![buildpass.png](/2024/usharp-asmdef/buildpass.png)

推測だが、これまで`/Package` 側で U# Script を書いたことがなかったので引っかからなかったと思われる。
