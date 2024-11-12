+++
title = "VRChatでボードゲームを作る"
date = 2024-11-12
draft = true
[taxonomies]
tags = ["tech", "VRChat", "Unity", "Udon", "UdonSharp", "C#", "Game"]
+++

掲題の通り、VRChat 上で動作するボードゲームを先日公開した。自分自身の備忘録また同様の制作を行いたく情報を探している人向けの制作記録として機能することを願って作業記録を残すことにする。

- [[無料] UdonKnucklebones [UdonChips 対応] - Booth](https://wipeseals.booth.pm/items/6278798)
- [wipeseals/UdonKnucklebones - GitHub](https://github.com/wipeseals/UdonKnucklebones)

![ss](/2024/create-vrchat-game/2024-11-10-170028.png)

---

## 作るもの

Knucklebones（ナックルボーン）と呼ばれるボードゲームである。本家は[Cult of the Lamb](https://store.steampowered.com/app/1313140/Cult_of_the_Lamb/?l=japanese) 作中に登場するゲームであり、今回の制作は非公式ファンアートの位置づけである。

サイコロを振って並べるという単純明快なルールでありながら、駆け引き要素がよくできたゲームである。（高得点を狙うほど、相手から一網打尽にされるリスクが上がっていくリスクとリターンがある）
もちろん、Knucklebones だけでなくゲーム自体の完成度もすばらしいのでぜひ遊んでみてほしい。

## 制作手順

おおよそ以下の手順で進めた。あくまでいち参考として見ていただけるとありがたい。

1. ゲームシステム試作
2. モデリング
3. VCC 配布可能な Unity Project 作成
4. モデルの Unity Setup
5. ゲームロジック 実装
6. 動作確認
7. サンプルワールド作成
8. 配布準備

<!-- more -->

## ゲームシステム試作

まずは Unity 等々は抜きにして、ただのコンソールアプリケーションとして Knucklebones を実装した。 必須の手順ではないが、特に実装に慣れていない・対象のゲームを作ったことがない人に関しては特におすすめしたい。
User が操作する部分を標準入力、User に見せるべき情報を標準出力で出す程度のものでも以下の収穫がある。

- Unity や同期特有の話を抜きにした実現可能性の確認
- ゲームロジックの理解が深まる
- 表示機として必要な要素の洗い出し
- ユーザー操作が必要な要素の洗い出し

自分が作ったものも相当簡素なものだが、ターン制であること、特定計算のロジック確認、プレイヤーがサイコロを振るフェーズと置くフェーズに分かれていること、などが確認できた。

[wipeseals/knucklebones_dice_game - GitHub](https://github.com/wipeseals/knucklebones_dice_game)

```log
$ dotnet run
Welcome to KnuckleBones!
Turn: 1 Player: 0
=========================================
Player1 Score: 0
Player2 Score: 0
=========================================
 Player 1  | Player 2
===========|===========
 0 | 1 | 2 | 0 | 1 | 2
===========|===========
   |   |   |   |   |   |
   |   |   |   |   |   |
   |   |   |   |   |   |
Player 1 rolled a 5.
Enter the column number to place your piece:

...

Player 1 rolled a 2.
Enter the column number to place your piece:
2
Turn: 20 Player: 1
=========================================
Player1 Score: 25
Player2 Score: 38
=========================================
 Player 1  | Player 2
===========|===========
 0 | 1 | 2 | 0 | 1 | 2
===========|===========
 2 | 4 | 2 | 4 | 6 | 4 |
 1 | 1 | 3 | 5 | 2 | 6 |
 1 | 1 | 2 | 6 |   | 5 |

Player 2 wins!
```

手慣れている好きな言語で実装するとよいが、C#で書いておくと Udon#に持ち込んだときに多少流用できるのでお得かもしれない。

## モデリング

ゲームシステムを試作すればおおよそどのような UI/UX にするかはイメージが付いている...がデザインについてはそうでもない。
サイコロを転がすこと、並べることができるコンパクトなデザインを目指した。サイコロは Unity 中で複製すればよいので 1 個だけ作成。

最初にサイコロを並べて、後から外形を作った感じで仕上げた。

![ss](/2024/create-vrchat-game/2024-09-28-225833.png)

デザインについては小綺麗にしたかったこともあり、大理石でホテルの受付においてあっても違和感のなさそうなものを目指した。
UV 展開を済ませてからのテクスチャ調整を行っている。

![ss](/2024/create-vrchat-game/2024-09-29-150954.png)

ここまでできたらテクスチャとモデルを Export して Unity に持ち込む準備を済ませておく。
また、表示上シェイプキーを使用する場合はこの際に入れておくと後が楽かもしれない（今回は使わなかった）

## VCC 配布可能な Unity Project 作成

最近の VRChat のライブラリ等々の配布には VCC (VRChat Creator Companion) を使用することができる。
もちろん従来通り必要なファイルを.unitypackage にまとめてもよいのだが、今回は無料公開配布することや修正のたびに.unitypackage を再生成するのが面倒くさかった、などの理由から VCC での配布前提でプロジェクトを作成した。

以下テンプレートパッケージを派生させ、Package/Unity version の更新と自分のプロジェクト向けに名称の変更などを施した。
GitHub Actions を使っての VCC 配布用データのビルド・配布用ページの作成を自動で行ってくれる便利テンプレートとなっている。

[vrchat-community/template-package - GitHub](https://github.com/vrchat-community/template-package)

以下ページが自動生成されている、ありがたい。

<https://wipeseals.github.io/UdonKnucklebones/>

![ss](/2024/create-vrchat-game/2024-11-11-004009.png)

## モデルの Unity Setup

Blender でモデリング・テクスチャリングしたものを取り込んだだけでは、まだゲームとして使うためには不十分である。
主に見た目や挙動の話だが、見た目の質感を整え、プログラムから制御される動き（アニメーション）はアニメーションクリップ+コントローラで制御できるようにしておく必要がある。

### マテリアル

インポートしたモデルに割り当てる質感を設定する。今回は殆ど一緒にエクスポートしてきたテクスチャを設定していくだけで済んだ。以下は Unity 上での見た目である。

![ss](/2024/create-vrchat-game/2024-09-29-155212.png)

今回は使わなかったが、プログラム中からマテリアルもといシェーダーのパラメータを弄って表示をいじるのであれば、それ用の Shader とマテリアルも準備しておく。

### アニメーション

今回はサイコロを振る動作は物理演算ベースで行いたかったため、アニメーションクリップはサイコロの配列を表示機として使う部分だけとなった。
サイコロの各面を上に向けるアニメーション、表示・非表示と同じ目が複数あったときのマーカー表示などのアニメーションクリップを作成した。

そして、プログラムから変数をいじったときに所定の状態遷移によって表示を切り替えられるようにアニメーションコントローラを作成する。
Play mode であれば値を弄くって挙動確認できるので、案外触りながら覚えたほうが良いかもしれない。以下図はサイコロの面を決めるレイヤだが、見た目がエモい

![ss](/2024/create-vrchat-game/2024-09-29-215021.png)

### コライダー等その他 UI

サイコロ自体に Box Collider を当てたほか、サイコロを振る際に吹き飛ばないようにするため等透明な壁としてのコライダーと、ユーザーが置き場所を決めたりサイコロを振るときの UI としてのコライダーを設置した。
以下 gizmo 表示の緑線がそれに当たる。
この他にも表示機としての Canvas, text や転がしたときの音の再生ソースなどもこのあたりで設置しておく。

![ss](/2024/create-vrchat-game/2024-10-15-220924.png)

## ゲームロジック 実装

いよいよ本命のゲームロジック実装に移る。ここまでの手順でプログラムから見た目をいじったりユーザー入力を受け取る部分は大方完成しておくとよいが、後から変えたくなった場合も適宜更新していけば良い。

### 同期について

既に公式情報や有志によるまとめが多数存在していて非常にありがたい環境になっている（感謝！）。細かい部分は紹介しないが、ざっくりいうと各ユーザーがワールド中で見えている情報はあくまで現在の変数を下に描画した結果に過ぎず、ゲームの進捗等に当たる変数を明示的に同期（値をワールドにいるメンバに共有）してあげないと、同じ景色は見えない設計になっている。

VRChat ユーザであれば、ローカルのミラーをイメージしてもらうと良いかも。ミラーのオンオフや座標を他プレイヤーに共有していないので自分にしか見えていないオブジェクトになっている。

### 同期の戦略について

Udon を用いた変数同期には、None（同期なし）、Continuous（連続同期）、Manual（マニュアル同期）のいずれかがよく用いられる認識。
難しいことを考えないのであれば Continuous にしておけば、UdonSynced を設定した変数は不定期に同期してくれる。後は変数を書き換えるユーザーが Owner （変数の所有者） を取得しておけば良い。

ただ、今回はターン制で値書き換えのタイミングがはっきりしていたこと、同期するタイミングがわからないのが気持ち悪かったことから Manual を採用した。

[UdonSharp#UdonSynced - docs.vrchat.com](https://udonsharp.docs.vrchat.com/udonsharp/#udonsynced)

### Late-Joiner 対応について

後からワールドに入ってきた人に進捗を同期する対応のこと。Continuous を採用していれば自然に変数の同期が終わり `FieldChangeCallback` 呼び出しで見た目が更新されていけば特別困ることはないはず。

今回は Manual 同期であることとあとから来た人が明示的に同期するトリガが欲しかったので、 Owner 宛に同期イベント発行の要求を飛ばし明示的に変数同期と UI 更新イベント発行するようにしている。

```log
[Owner]                 [Late-Joiner]
  |
  |
  |                     (OnPlayerJoined)
  |                            |
  |<------Request Sync---------|         *NetworkCustomEvent
  |
  |------Sync Vars------------>|         *RequestSerialization
  |------Update UI------------>|         *NetworkCustomEvent
                               |
                          (Sync Done)
```

### Knucklebones ゲームロジック

まずゲームの進行については、Player 参加待ち → サイコロを振る → サイコロを置く → 特定計算を行う → （サイコロを振るに戻る or ゲーム終了） だったので、これに対応する enum を定義し、enum に応じたゲーム進行を実装した。

```cs
    public enum GameProgress : int
    {
        /// <summary>
        /// 初期状態。IsActive=falseで開始されたケースでOnEnable/OnDisable/Startが呼ばれない
        /// OnPlayerJoinedで初期化する
        /// </summary>
        Initial = 0,
        /// <summary>
        /// Player1/2の参加待ち
        /// </summary>
        WaitEnterPlayers,
        /// <summary>
        /// ゲーム開始。再戦用に設けた状態
        /// </summary>
        GameStart,

        /// <summary>
        /// Player1のサイコロ転がし待ち
        /// </summary>
        WaitPlayer1Roll,
        /// <summary>
        /// Player1のサイコロ転がし中
        /// </summary>
        Player1Rolling,
        /// <summary>
        /// Player1のサイコロ配置待ち
        /// </summary>
        WaitPlayer1Put,
        /// <summary>
        /// Player2のサイコロ計算待ち。ついでにサイコロの前詰めアニメーションも行う
        /// </summary>
        WaitPlayer1Calc,

        /// <summary>
        /// Player2のサイコロ転がし待ち
        /// </summary>
        WaitPlayer2Roll,
        /// <summary>
        /// Player1のサイコロ転がし中
        /// </summary>
        Player2Rolling,
        /// <summary>
        /// Player2のサイコロ配置待ち
        /// </summary>
        WaitPlayer2Put,
        /// <summary>
        /// Player2のサイコロ計算待ち。ついでにサイコロの前詰めアニメーションも行う
        /// 次の遷移先はWaitPlayer1Roll or GameEnd
        /// </summary>
        WaitPlayer2Calc,

        /// <summary>
        /// ゲーム終了
        /// </summary>
        GameEnd,
        /// <summary>
        /// ゲーム中断
        /// </summary>
        Aborted,
        /// <summary>
        /// 設定エラー
        /// </summary>
        ConfigurationError,
    }
```

また、Udon での多次元配列のサポートが怪しかった（今は平気？）ことが頭をよぎったので、u64 (C# の ulong) の数値を bitfied 単位で扱い、4bit ごとにサイコロ値を格納するラッパー実装を行った。
そのため、UdonSynced で同期する盤面は ulong の変数となっていて、盤面自体は DiceArrayBits class で取得・更新する方式となった。

同期変数

```cs
        /// <summary>
        /// Player1のサイコロ配置(生データ)
        /// </summary>
        [UdonSynced(UdonSyncMode.None), FieldChangeCallback(nameof(Player1DiceArrayBits))]
        public ulong _player1DiceArrayBits = 0;

        /// <summary>
        /// Player1のサイコロ配置(生データ)
        /// </summary>
        public ulong Player1DiceArrayBits
        {
            get => _player1DiceArrayBits;
            set
            {
                _player1DiceArrayBits = value;
            }
        }

        /// <summary>
        /// Player2のサイコロ配置(生データ)
        /// </summary>
        [UdonSynced(UdonSyncMode.None), FieldChangeCallback(nameof(Player2DiceArrayBits))]
        public ulong _player2DiceArrayBits = 0;

        /// <summary>
        /// Player2のサイコロ配置(生データ)
        /// </summary>
        public ulong Player2DiceArrayBits
        {
            get => _player2DiceArrayBits;
            set
            {
                _player2DiceArrayBits = value;
            }
        }
```

サイコロの値取得・更新関連

この他にも配置したときの同じ目を消す処理や特定計算なども実装してある。

```cs
        /// <summary>
        /// サイコロの配置から通し番号を取得する。LSBから列0行0, 列0行1, 列0行2, 列1行0, ... となる
        public static int GetDiceIndex(int col, int row) => col * NUM_ROWS + row;

        /// <summary>
        /// サイコロの配置からサイコロの値を取得する
        /// </summary>
        public static int GetDice(this ulong bits, int col, int row)
        {
            var index = GetDiceIndex(col, row);
            var bitOffset = index * DICE_BIT_WIDTH;

            return (int)((bits >> bitOffset) & DICE_BIT_MASK);
        }

        /// <summary>
        /// サイコロの配置を更新した値を返す
        /// </summary>
        public static ulong PutDice(this ulong bits, int col, int row, int value)
        {
            var index = GetDiceIndex(col, row);
            var bitOffset = index * DICE_BIT_WIDTH;

            // 一度クリアしてからセットする
            bits &= ~(DICE_BIT_MASK << bitOffset);
            bits |= ((ulong)value & DICE_BIT_MASK) << bitOffset;

            return bits;
        }
```

### UdonChips 対応について

TODO:

## 動作確認

### 単体テスト

TODO:

### Unity 上で動作確認

TODO:

## サンプルワールド作成

TODO:

## 配布準備

TODO:

## 総括

TODO:
