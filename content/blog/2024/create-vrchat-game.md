+++
title = "VRChatで動くボードゲームを作る"
date = 2024-11-14
[taxonomies]
tags = ["tech", "VRChat", "Unity", "Udon", "UdonSharp", "C#", "Game"]
+++

掲題の通り、VRChat 上で動作するボードゲームを先日公開した。自分自身の備忘録また同様の制作を行いたく情報を探している人向けの制作記録として機能することを願って作業記録を残すことにする。

- [[無料] UdonKnucklebones [UdonChips 対応] - Booth](https://wipeseals.booth.pm/items/6278798)
- [wipeseals/UdonKnucklebones - GitHub](https://github.com/wipeseals/UdonKnucklebones)

![ss](/2024/create-vrchat-game/2024-11-10-170028.png)

---

## 作るもの

Knucklebones（ナックルボーン）と呼ばれるボードゲームを作ることにした。本家は[Cult of the Lamb](https://store.steampowered.com/app/1313140/Cult_of_the_Lamb/?l=japanese) 作中に登場するゲームであり、今回の制作は非公式ファンアートの位置づけである。

サイコロを振って並べるという単純明快なルールでありながら、駆け引き要素がよくできたゲームである。（高得点を狙うほど、相手から一網打尽にされるリスクが上がっていく）
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

自分が作ったものも相当簡素なものだが、ターン制であること、得点計算のロジック確認、プレイヤーがサイコロを振るフェーズと置くフェーズに分かれていること、などが確認できた。

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

今回はサイコロを振る動作は物理演算ベースで行いたかったため、アニメーションクリップはサイコロの配列を表示機として使う部分と、後述するがサイコロを転がす前の回転表示だけとなった。
サイコロの各面を上に向けるアニメーション、表示・非表示と同じ目が複数あったときのマーカー表示などのアニメーションクリップを作成した。

そして、プログラムから変数をいじったときに所定の状態遷移によって表示を切り替えられるようにアニメーションコントローラを作成した。
Play mode であれば値を弄くって挙動確認できるので、案外触りながら覚えたほうが良いかもしれない。以下図はサイコロの面を決めるレイヤだが、見た目がエモい。

![ss](/2024/create-vrchat-game/2024-09-29-215021.png)

### コライダー等その他 UI

サイコロ自体に Box Collider を当てたほか、サイコロを振る際に吹き飛ばないようにするため等透明な壁としてのコライダーと、ユーザーが置き場所を決めたりサイコロを振るときの UI としてのコライダーを設置した。
以下 gizmo 表示の緑線がそれに当たる。
この他にも表示機としての Canvas, text や転がしたときの音の再生ソースなどもこのあたりで設置しておく。

![ss](/2024/create-vrchat-game/2024-10-15-220924.png)

## ゲームロジック 実装

いよいよ本命のゲームロジック実装に移る。ここまでの手順でプログラムから見た目をいじったりユーザー入力を受け取る部分は大方完成しておくとよいが、後から変えたくなった場合も適宜更新していけば良い。

### 同期について

既に公式情報や有志によるまとめが多数存在していて非常にありがたい環境になっている（感謝）。細かい部分は紹介しないが、ざっくりいうと各ユーザーがワールド中で見えている情報はあくまで現在の変数を下に描画した結果に過ぎず、ゲームの進捗等に当たる変数を明示的に同期（値をワールドにいるメンバに共有）してあげないと、同じ景色は見えない設計になっている。

VRChat ユーザであれば、ローカルのミラーをイメージしてもらうと良いかも。ミラーのオンオフや座標を他プレイヤーに共有していないので自分にしか見えていないオブジェクトになっている。

### 同期の戦略について

Udon を用いた変数同期には、None（同期なし）、Continuous（連続同期）、Manual（マニュアル同期）あたりがある。
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

まずゲームの進行については、Player 参加待ち → サイコロを振る → サイコロを置く → 得点計算を行う → （サイコロを振るに戻る or ゲーム終了） だったので、これに対応する enum を定義し、enum に応じたゲーム進行を実装した。

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

この他にも配置したときの同じ目を消す処理や得点計算なども実装してある。

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

### サイコロを振る動作について

サイコロを振る動作はできるだけランダム要素と自然な挙動を重視したく、設計当初から物理演算任せにしようとは考えていた。
ただ、ユーザー自身に振る動作をするとイカサマ云々が面倒になるので、あくまでタイミング制御のみを担うことにしている。

実装時に一つ問題があり、物理演算を行う物体に Animation Controller から干渉すると期待通りの挙動にならなず、変な水平移動が出たりと物理演算側が破綻してしまう。（よく考えれば当たり前の気がしてくるが...）
これの対策に、振るまでの間中空で回転アニメーションを再生するサイコロと、振った瞬間の位置・向きを転写しつつランダムな方向に吹き飛ばして物理演算で転がりを表現する 2 種類のサイコロをユーザー見えには自然につながるように組み合わせた。

以下 gif で座標軸の gizmo が表示されているのは転がっているサイコロ。ひたすら回転しているサイコロと表示を切り替えることで、物理演算との組み合わせにできた。

![movie](/2024/create-vrchat-game/knucklebones-test3.gif)

サイコロの向いている面は、CustonEventDelaySeconds で遅延させたイベントで何度か監視し、面の法線方向のベクトルと Y+ベクトルの角度が最も小さい面を採用とした。
LINQ が使えないので若干不格好だが、愚直でシンプルな実装になっていると思われる。

```cs
            // DiceForRollの向き情報を取得
            var forward = DiceForRoll.gameObject.transform.forward;
            var up = DiceForRoll.gameObject.transform.up;
            var right = DiceForRoll.gameObject.transform.right;
            var back = -forward;
            var down = -up;
            var left = -right;

            // Vector3.upとの角度が最も小さいものが上面
            // anglesのindexはFaceDirectionと対応
            var angles = new[] {
                Vector3.Angle(forward, Vector3.up),
                Vector3.Angle(back, Vector3.up),
                Vector3.Angle(left, Vector3.up),
                Vector3.Angle(right, Vector3.up),
                Vector3.Angle(up, Vector3.up),
                Vector3.Angle(down, Vector3.up),
            };

            // 最小値のインデックスを取得
            var minIndex = 0;
            for (var i = 1; i < angles.Length; i++)
            {
                if (angles[i] < angles[minIndex])
                {
                    minIndex = i;
                }
            }
```

動きの収束には `Rigidbody.IsSleeping` と、一定時間のタイムアウトのどちらかを満たした時点を採用した。
また、高速で弾き出した際に壁を貫通する恐れがあったので、速いオブジェクト対静的オブジェクトの衝突と考え Collision Detection を Continuous に設定してある。

### UdonChips 対応について

こちらは作り始めた時点でぜひ実装したかった要素である。一方で、VCC 配布の都合上 asmdef で分けていることで頭を悩ませた点でもある。
UdonChips の実装は、シーン中に置いてある UdonChips という名称の GameObject を取得し、これの money を操作することで実現している。

この UdonChips Object の class には asmdef の定義はなく、 `Assembly-CSharp > UCS > UdonChips` の定義になっている。
そのため、`me.wipeseals.udon-knucklebones` package からはそのままでは見えず、`Assembly-CSharp` を参照するかこれ自体に属する必要があった。

対応としては `UdonKnucklebones` を継承可能なクラスとして実装しておき、別途 `Assembly-CSharp` に属する `UdonKnucklebonesSupportUdonChips` class を作り、ここから UdonChips 関連の操作だけを加える方法を取っている。
UdonChips がない環境でコンパイルエラーにならないように、unitypackage で配布パッケージに同梱してある。

```log

my package            |    Assembly-CSharp
----------------------|----------------------
                      |
                      |         UCS.UdonChips
                      |             ^
                      |             |
                      |             |
                      |           [参照]
                      |             |
                      |             |
UdonKnuckebones<----[継承]---UdonKnuckebonesSupportUdonChips
                      |
                      |
                      |
```

UdonKnuckebonesSupportUdonChips class は表示と UdonChips 更新関連の処理しか書かないで済むようになっている。継承使えてよかった。

```cs
public class UdonKnucklebonesSupportUdonChips : UdonKnucklebones
{
    /// <summary>
    /// 取引されたならtrue
    /// </summary>
    bool _isPaid = false;
    /// <summary>
    /// 取引レート
    /// </summary>
    float _rate = 0.0f;
    /// <summary>
    /// 取引されたUdonChipsの金額
    /// </summary>
    float _applyMoney = 0.0f;

    /// <summary>
    /// システムメッセージを取得。UI反映用のメッセージ
    /// </summary>
    /// <returns></returns>
    public override string GetSystemMessage()
    {
        //...
    }

    /// <summary>
    /// 所持金を取得し同期変数に設定
    /// </summary>
    public override void OnUpdateCurrentUdonChips()
    {
        //...
    }

    /// <summary>
    /// 勝敗の金額を反映。ローカル処理
    /// </summary>
    public override void OnApplyUdonChips()
    {
         //...
    }
}
```

## 動作確認

### 単体テスト

ゲームの根幹に関わる部分で、特定の盤面でないと発生しないような不具合はできるだけ論理検証で潰しておきたかった。
幸いにもサイコロの配置の更新や点数計算などは u64 のビット演算として Utility に切り出していたので、これの論理検証を実装してある。

本当は PlayMode Test 込で網羅性の高いものにしたかったが、品質と時間の妥協ラインがこのあたりだったのでひとまず自分が必要だと思った範囲に絞っている。
(4bit ごとに 1 個割り当てているおかげで、hex 表示すると盤面が見えやすくて助かった)

```cs
        [TestCase(0x00000000_00000000UL, 0, ExpectedResult = 0)]
        [TestCase(0x00000000_00000001UL, 0, ExpectedResult = 1)]
        [TestCase(0x00000000_00000011UL, 0, ExpectedResult = (1 + 1) * 2)]
        [TestCase(0x00000000_00000111UL, 0, ExpectedResult = (1 + 1 + 1) * 3)]
        [TestCase(0x00000000_00001111UL, 0, ExpectedResult = (1 + 1 + 1) * 3)]
        [TestCase(0x00000000_00011111UL, 0, ExpectedResult = (1 + 1 + 1) * 3)]
        [TestCase(0x00000000_00111111UL, 0, ExpectedResult = (1 + 1 + 1) * 3)]
        [TestCase(0x00000000_01111111UL, 0, ExpectedResult = (1 + 1 + 1) * 3)]
        [TestCase(0x00000000_11111111UL, 0, ExpectedResult = (1 + 1 + 1) * 3)]
        [TestCase(0x00000001_11111111UL, 0, ExpectedResult = (1 + 1 + 1) * 3)]
        [TestCase(0x00000001_11111112UL, 0, ExpectedResult = (1 + 1) * 2 + 2)]
        [TestCase(0x00000001_11111116UL, 0, ExpectedResult = (1 + 1) * 2 + 6)]
        [TestCase(0x00000001_11111156UL, 0, ExpectedResult = 1 + 5 + 6)]
        [TestCase(0x00000001_11111456UL, 0, ExpectedResult = 4 + 5 + 6)]
        [TestCase(0x00000002_22333444UL, 0, ExpectedResult = (4 + 4 + 4) * 3)]
        [TestCase(0x00000002_22333444UL, 1, ExpectedResult = (3 + 3 + 3) * 3)]
        [TestCase(0x00000002_22333444UL, 2, ExpectedResult = (2 + 2 + 2) * 3)]
        public int GetColumnScoreTest(ulong initialBits, int col)
        {
            return DiceArrayBits.GetColumnScore(initialBits, col);
        }

        [TestCase(0x00000000_00000000UL, ExpectedResult = 0)]
        [TestCase(0x00000000_00000001UL, ExpectedResult = 1)]
        [TestCase(0x00000000_00001001UL, ExpectedResult = 1 + 1)]
        [TestCase(0x00000000_01001001UL, ExpectedResult = 1 + 1 + 1)]
        [TestCase(0x00000000_21021021UL, ExpectedResult = 1 + 2 + 1 + 2 + 1 + 2)]
        [TestCase(0x00000003_21321321UL, ExpectedResult = 1 + 2 + 3 + 1 + 2 + 3 + 1 + 2 + 3)]
        [TestCase(0x00000006_66555444UL, ExpectedResult = (4 + 4 + 4) * 3 + (5 + 5 + 5) * 3 + (6 + 6 + 6) * 3)]
        public int GetTotalScoreTest(ulong initialBits)
        {
            return DiceArrayBits.GetTotalScore(initialBits);
        }
```

### 実際の動作確認

まずは Unity 上で問題なく動作することを確認した。その後、VRChat Client を複数起動してのデバッグで同期関連を見ている。
観点としては以下のようにネットワーク周りの不具合と、VR 環境特有の使い勝手の悪さがないかなどを気にしていた。

- ネットワーク越しに正常に動作するか
- 途中で Player の片方・両方が離脱したときに操作不能にならないか
  - （どちらかが抜けた時点で、試合を Abort 状態にして初期状態に戻している）
- 後から入ってきた Player に状況が見えるか
- 後から入ってきた Player が参加して Play 可能か
- 極端に重くなるケースや、どちらかの Player にだけ見えていない現象などがないか
- 特定のやり取りは正常に行われるか
- Interact する距離が遠すぎる・短すぎる、変な位置に Player と衝突するコライダーがないか

クライアント増やしたり減らしているしている図

![ss](/2024/create-vrchat-game/2024-10-15-220508.png)

## サンプルワールド作成

ここは必須ではなかったが突然気乗りしたので作った。コンセプトはカラオケの居抜きのような狭い空間でゲームだけをするスペースのイメージ。
ゲーム本体と同じく Blender で制作し、Unity で細かい見た目の調整を施した。まっさらなサンプルワールドよりかはおしゃれになった。

![ss](/2024/create-vrchat-game/2024-10-10-001643.png)

## 配布準備

VCC で配布をするものの、Package 関連の主な流通は Booth もしくは Vket という認識なので、これ向けにもなるように README.md に説明文章を作成した。
また、ワールド制作者がゲーム自体の説明を書かなくて済むように、ゲーム説明用のパネルのようなものも作成・同梱してある。

![ss](/2024/create-vrchat-game/UdonKnucklebones-Manual-3.png)

後は Github Actions で最新版をリリースし、Booth に説明等々を乗せてリリースして完了。あとは変なバグがないことを祈る。

## 総括

だいぶ駆け足で包括的な説明にはなっていないが、書き残しておきたかった要素要素には触れることできたのではないかと思う。

Console UI でゲームを作るのに対し考えることが多い分、創意工夫できる点が多く実際に動いたり遊んでもらえたときにはかなり嬉しい。
何より、Platform が用意されているので遊んでもらうまでの敷居が低いので、テストプレイなどもお願いしやすい。（ありがとうございました）

ゲーム自体の完成度や考え方については、もう少し数をこなしたりユーザーの操作体験を重視した複雑なものを作ると見えてくるものがあるかもしれない。
