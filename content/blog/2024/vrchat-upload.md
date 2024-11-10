+++
title = "2024年5月時点 VRChat Avatar Upload環境memo"
date = 2024-05-04
[taxonomies]
tags = ["tech", "vrchat", "unity", "memo"]
+++

Zenn 執筆環境構築のテスト記事も兼ねてアバター導入環境について言及。多分もっと丁寧な記事があると思うので備忘録程度。

## 必要な環境とその手順

- VRChat アカウント
  - VRChat には [Trust Rank](https://docs.vrchat.com/docs/vrchat-safety-and-trust-system) という概念があり、作りたてのアカウントでは Visitor となっている
  - やり込んで Rank を上げるか、課金することで New User 以上の権限を持つアカウントになることで Upload 可能
    > The transition between “Visitor” and “New User” is a special one — when a Visitor becomes a New User, they gain the ability to upload content to VRChat as long as they’re using a VRChat account. Users receive a notification when they have passed this rank, and are directed to the VRChat documentation page to get started with creating content.
- Unity Hub
  - Unity は複数バージョンを共存させることがあり、これの管理に(?)Unity Hub を使用する
  - [Unity.com](https://unity.com/ja/download) から DL してセットアップしておく、また起動後アカウントやライセンス認証を求められる場合は Unity のアカウントも取得しておく
- Unity 2022.3.6f1
  - [VRChat 公式によるとこのバージョンが推奨のため](https://creators.vrchat.com/sdk/upgrade/current-unity-version/)、 Unity Hub 導入後に VRChat の現時点での最新バージョンをいれる
    - [unityhub://2022.3.6f1/b9e6e7e9fa2d](unityhub://2022.3.6f1/b9e6e7e9fa2d)
  - (筆者は 2019 を使ったことあるが 2022 が初なのでせっかくなので本記事を書き残す) > The current Unity version used by VRChat is 2022.3.6f1.
    If you have Unity Hub installed, you can click this link to install the correct version of Unity. 2022.3.6f1 is also available in the Unity editor release archive.
- VRChat Creator Companion (VCC)
  - VRChat が Unity 向けに提供している Avatar, World 向けの SDK などを手軽に導入・管理(?)するためのツール
  - VRChat アカウント取得後なら [https://vrchat.com/home/download](https://vrchat.com/home/download) から DL できる
- (VCC 代替) ALCOM
  - anatawa12 さんが作成している VCC 互換ツール。 [vrc-get/vrc-get - GitHub](https://github.com/vrc-get/vrc-get) の GUI FrontEnd
  - 使ってみた感じ本家よりかなり高速で快適に使用できているので、オススメしたい。以後の説明で VCC を使い箇所は ALCOM で読み替え可
  - [https://vrc-get.anatawa12.com/alcom/](https://vrc-get.anatawa12.com/alcom/) から DL 可能
- 何らかの画像エディタ
  - アバターの改変用。筆者はクリスタ常用 (だが、どちらかといえば PSD 同梱してくださる方が多い印象)

<!-- more -->

## Avatar Project で毎回使っているライブラリ群

VRChat 公式が用意している SDK の他に、Shader を始め付随して使用することで改変がやりやすくなったりリッチな見た目にできたりというものがある。
2024 年現在では Booth で販売されている多くのアセットも依存関係を持っており、以下ツールはほぼ必須と考えている。（開発ありがとうございます）

ライブラリの導入には Unity Package 形式(.unitypackage)をプロジェクトに展開してインストールする方法と、VCC にライブラリの URL (VPM) を事前登録しておいて都度最新を取得する方法がある。
後者の手順が提供されている場合、基本そちらを利用する方を強く推奨する。
_アップデートが容易に行える、依存関係のあるバージョン解決など VCC を使ったプロジェクト管理をある程度やったときにメリットが見えてくると思われる。_

Booth で提供される有償アセットなどはどうしても前者になってしまうがこれは仕方ない。
**注意として VCC を使って導入したライブラリ群を Unity Package で再度導入しない方が良い。 UnityPackage を開いたときに新規追加・変更されるファイル一覧が出る時に確認した方が良い。**

- [lilToon](https://lilxyzw.github.io/lilToon/#/)
  - アバター向けのシェーダー。他にもいくつかあるが使いやすく最近はこれを前提にしているものが多い。 VCC のボタンから追加できる
- [lilAvatarUtils](https://github.com/lilxyzw/lilAvatarUtils)
  - lilToon 作者の lilxyzw さんが提供されている Utility。アバターに使用している画像一覧とそのサイズを表示したり、Fallback（軽量表示時）の見た目を確認したりできる
  - lilToon 導入時に [lilxyzw さんの VPM repo を登録済なら](https://github.com/lilxyzw/vpm-repos) VCC から導入可
- [MA: Modular Avatar](https://modular-avatar.nadena.dev/ja)
  - 改変したことがないとピンとこないかもしれないが、髪型・衣装などの変更を簡単に行うことができるツール
    :::details MA 以前は?
  - (専門ではないのでざっくりした説明になるが) アバターの各部位や衣装などのメッシュをボーンに追従して動かすためには、メッシュの各頂点がどのボーンに対しどの程度連動して動くか、という情報を持っている
    - アバター自作などでよく聞くウェイトペイント (のうち、Humanoid Bone への追従を塗る場合) がこの設定であり、ウェイト=0 無関係、ウェイト=1 完全追従 といった具合
  - 厄介なのが、アバターの素体も衣装も同じ Humanoid の Bone 構成に追従して動くが、もとのアバターのウェイト追従先 Bone と改変用の衣装のウェイト追従先 Bone が異なる Object になっている
    - 衣装製作者がアバター自体の素体を同梱しないと回避不能で、この手順は販売観点では難しそう (他人が作成された有償アセットの一部ないし全部を自身のアセットに組み込むことになる)
  - Modular Avatar は Unity 上で Play 時、Upload 時にこの同じ Bone 構成になっている Object 同士を合成してくれるので、着せたり脱がせたりが容易にできるようになる
    - もとのアバターの Bone 構造の各 Object の直下に、着せたい衣装などの Bone Object を突っ込む（Object が親子になっていれば親追従）ことで解決していた様子
      - この方式では特に着せた衣装を剥がすのが大変になる。Prefab Variant 使いこなしていればうまいことできる気もするが筆者は余り実績なし
      - 別案としては Blender でやる方法など。Unity 外なので Blender 自身をある程度使える状態でないと敷居は高いかも
    - Modular Avatar は Bone 構造の合成以外にも便利機能を用意している。また、Play/Upload 時の処理フローを [Non-Destructive Modular Framework](https://github.com/bdunderscore/ndmf) で提供しており、ツール間の干渉も考慮して設計されていて大変ありがたい
  - 上記の説明では Skinned Mesh Renderer と Bone の関係性に触れたが、Animation や ExpressionParameter なども同様にアバターと改変衣装の合成が必要。ますます剥がすのが大変になる...
    :::
- [AAO: Avatar Optimizer](https://vpm.anatawa12.com/avatar-optimizer/ja/)
  - アバター軽量化のための Utility 群。Unity 上で頂点を消したりオブジェクトを合成したりなどはできないことはないが非破壊で行えるというのはかなり魅力的 - 服で見えなくなる頂点を消す、使われていない Bone を消す、不要な BlendShape を消す、PB をまとめる...など - 手軽で非破壊なのでとても使いやすい。特に [Trace And Optimize](https://vpm.anatawa12.com/avatar-optimizer/ja/docs/reference/trace-and-optimize/), [Remove Mesh By BlendShape](https://vpm.anatawa12.com/avatar-optimizer/ja/docs/reference/remove-mesh-by-blendshape/) はよくお世話になっています
    :::details 軽量化は必要?
  - やらなくても動きはするが、同じワールドにいる人にリソースを多く消費させることになるので、可能なら軽量化対策はやった方が良さそう - もし軽量化が難しいのであれば、大勢が集まるイベントをはじめとした環境には余り持ち出さないなどを検討した方が良い - もちろん重たいアバターを非表示にするような自衛する仕組みはあるが、全員が使いこなせているとは限らないので
    :::
- [anatawa12's gist pack for Unity](https://vpm.anatawa12.com/gists/ja/)
  - AAO 作者の anatawa12 さんが公開されているツール群。
  - MA,AAO 実行後の Performance Rank 確認のために [Actual Performance Window](https://vpm.anatawa12.com/gists/ja/docs/reference/actual-performance-window/) を使用している
    - VRChat SDK 公式は MA,AAO などを実行前の状態でパフォーマンスランクを計算している様子
- [FaceEmo](https://suzuryg.github.io/face-emo/ja/)
  - アバターの表情を簡単に作成できるツール。表情は BlendShape を変更する AnimationClip を作り再生するという手順を何らかのトリガ(e.g. ハンドサイン)で行うという地味に手順が多い
  - FaceEmo ではトリガの設定、表情のプレビュー、撮影用に表情を固定するなど、Unity 特有の部分を隠蔽してくれる
- [VRChat Gesture Manager](https://github.com/BlackStartx/VRC-Gesture-Manager)
  - Unity 上でアバターの見た目や挙動を確認するツール。表情やアニメーションなどの動作確認のために毎回 VRChat を起動しなくて良くなる

## Project 作成から Upload まで

細かいコンポーネントの使い方は先述の公式 document にまとめてくださっているのでそちらを見るか、他記事などを参照すると良さそう。

- VCC 起動して Project 作成
- 使用するライブラリを VCC から追加
  - VRChat SDK - Base
  - VRChat SDK - Avatars
  - 先述のライブラリ
- Unity で開く
- Unity で購入したアバター、衣装などの Unity Package 群
  - VCC で導入したものは飛ばすこと。よくある例としては Shader が同梱されるケースが有る
- 以後はアバターや衣装の導入手順に従う。以下は例
  - 素体 Prefab か衣装を着ている default Prefab を Scene 上に配置
  - 毎回最初にやっている
    - Tools > Gesture Manager Emulator で Play 時に Gesture Manager を使えるようにする（Scene 中に Object 追加され、そこで操作可能）
    - AAO Trace and Optimize を Object Root に追加
      - 軽量化目的
    - MA Mesh Settings を Object Root に追加、両方の Target を Hips に設定
      - Anchor Override, Bounds 設定用
      - 明るさの計算位置を統一、また Culling (視界外と判定して描画 Skip する)範囲を統一
        - 既存の Mesh Object がどこに Anchor Override を向けているかで変えることもある
    - Tools > anatawa12's gist selector で ActualPerformanceWindow を追加
      - Play 時に （MA, AAO 実行後状態の） Performance Rank が表示されるようになる
  - 衣装を着せる
    - 衣装 Prefab を Scene 上に配置
    - 衣装 Prefab の Object を素体 Prefab の Object 直下にいれる
    - 右クリック Modular Avatar > Setup Outfit
      - MA Mesh Settings 設定は、"毎回やっている" の Mesh setting が追加済なら継承。そうでないなら Hips あたりに
    - 素体 Object、衣装 Object が重なって見えなくなる部分を AAO Remove Mesh by XXX (よく使うのは BlendShape)で消えるように設定
  - 髪型を変える
    - 元の髪型がある場合、Layer を Editor Only、IsActive=false (上部チェックボックスを外して非表示にする)
      - Scene 上の非表示 ≠Avatar に同梱しない、となっている (Animation から表示非表示切り替えられる)ため、Editor Only 指定で生成物から除外
      - もしくは金輪際使わない Mesh であれば Scene 上から削除する (Unpack されていないと無理かも? 余りやらないので詳細不明)
    - 着せ替えたい髪の Prefab を Scene に追加。素体 Prefab 直下に移動、髪の Object を選択肢 Add Component で MA Bone Proxy, MA Mesh Setting を追加
    - MA Bone Proxy でターゲットを Head に（Play 時に頭直下に追従させる）、Mesh setting の Anchor Override, Bounds を継承、もしくは Hips あたりに合わせる
  - 色改変等
    - Avatar に使われている Mesh の Material 上でパラメータ調整、もしくは元画像を編集するか
  - 毎回最後にやっている
    - Window > \_lil > AvatarUtils から lilAvatarUtils を表示
    - Textures で使われているテクスチャ一覧で色改変前後がダブっていそうなものや、不用意に解像度が高いものを調整
    - Lighting で明るさ表示が変になっていないものがないか確認、また Safety On にして表示が変にならないことを確認
      - 変になっている場合、Materials から使われている Material を順に確認し、基本設定 > VRChat > Custom Safety Fallback を設定
        - 負荷対策などで Shader をデフォルトに戻された場合の表示結果。ある程度パラメータを明示できる
        - 自分の好みは、 顔・体・衣装などは Unlit Opaque、無理に表示しなくて良いものは Hidden に設定
          - 改変したのに色がもとに戻る場合、Shader のパラメータでのみ色変更されているなどが考えられる
            - 気になるなら焼き込みしてテクスチャとして色が変わった状態の画像を参照する状態にしておく
  - Upload
    - VRChat SDK > Show Control Panel で Upload 用のツールを表示。ログイン画面ではログインしておく
    - Avatar Info で名前や説明を記載、 **Visibility はアバターによるが、基本有償アセットであれば Private に設定**
    - Thumbnail はどんなアバターなのか分かる写真を取っておく。今の Scene 表示をそのまま撮影できる
    - Build & Test で Upload 前にローカルテストできる。VRChat 起動後 Avatar 一覧の Other にいる（おそらく他人には見えない）
    - 問題なければ Online Publishing のチェックを入れて同意後、Upload
  - その他
    - Avatar Project を git 管理する場合、以下考慮したほうが良い
      - .gitignore で中間生成物を始め不要なものを staging しないようにする
      - 有償アセットを公開しないようにすること(repo が Public だと公開されてしまう)
      - でかいファイルをそのまま staging すると push できなくなるので対策もしくは追加対象から除外する
        - 改変用 PSD がとても大きなファイルだったりする

だいぶ走り書きしたが、だいたいこんな感じでうまくいくと思われる。
