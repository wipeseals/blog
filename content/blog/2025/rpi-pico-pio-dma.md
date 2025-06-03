+++
title = "RPi Pico PIO + DMA 転送覚書 (MicroPython 版)"
date = 2025-06-03
[taxonomies]
tags = ["raspberrypi", "micropython", "pio", "dma", "embedded"]
+++

MicroPython を用いて Raspberry Pi Pico の PIO と DMA を使う方法についての覚書。それぞれの情報は入手できるが、TX_FIFO/RX_FIFO 両方に DMA する例がなかったので書き残す。

本内容は MicroPython のドキュメント並びに RP2040 の仕様書から抜粋した内容となっている。

## HW 上達成すべきこと

`PIOx_BASE` + `(T|R)XFy` のアドレスに読み書きを行うこと。

以下内容より PIO Address `x= (0|1)` と
`(T|R)XFy` のアドレス `y= (0|1|2|3)` を組み合わせて、PIO の TX_FIFO/RX_FIFO にアクセスする。

### `PIOx_BASE`

`PIOx_BASE` は、PIO の Base Address。 2.2. Address Map にて以下定義がなされている。

- `PIO0_BASE` = `0x50200000`
- `PIO1_BASE` = `0x50300000`

### `(T|R)XFy`

`(T|R)XFy` は、PIO の TX_FIFO/RX_FIFO のレジスタ。Chapter 3. PIO 3.7. List of Registers にて以下定義がなされている。

| Offset | Name   |
| ------ | ------ |
| 0x010  | `TXF0` |
| 0x014  | `TXF1` |
| 0x018  | `TXF2` |
| 0x01C  | `TXF3` |
| 0x020  | `RXF0` |
| 0x024  | `RXF1` |
| 0x028  | `RXF2` |
| 0x02C  | `RXF3` |

FIFO の空きに関連する情報は Offset 0x008 `FDEBUG` レジスタの `TXSTALL`, `TXOVER`, `RXUNDER`, `RXSTALL` 及び FLEVEL の `(T|R)Xy`から確認できる。

### DMA における FIFO Status の確認

DMA が一方的にデータ転送しても FIFO にデータがないのに読み出したり、あるのに書き込んで溢れてしまう。これを防ぐため、ペリフェラル（ここでは PIO）のペースでデータ転送を行えるようなしくみがあり、 Data Request (DREQ) と呼ばれる設定を行う。

2.5.3.1 System DREQ Table の設定があり、PIO に関連する内容は以下のようになっている。

`DREQ# = PIO# * 4 + (4 if RX else 0) + SM#`

| PIO | SM  | PORT | DREQ# | Description      |
| --- | --- | ---- | ----- | ---------------- |
| 0   | 0   | TX   | 0     | PIO0 SM0 TX DREQ |
| 0   | 0   | TX   | 1     | PIO0 SM1 TX DREQ |
| 0   | 0   | TX   | 2     | PIO0 SM2 TX DREQ |
| 0   | 0   | TX   | 3     | PIO0 SM3 TX DREQ |
| 0   | 0   | RX   | 4     | PIO0 SM0 RX DREQ |
| 0   | 0   | RX   | 5     | PIO0 SM1 RX DREQ |
| 0   | 0   | RX   | 6     | PIO0 SM2 RX DREQ |
| 0   | 0   | RX   | 7     | PIO0 SM3 RX DREQ |
| 1   | 0   | TX   | 8     | PIO1 SM0 TX DREQ |
| 1   | 0   | TX   | 9     | PIO1 SM1 TX DREQ |
| 1   | 0   | TX   | 10    | PIO1 SM2 TX DREQ |
| 1   | 0   | TX   | 11    | PIO1 SM3 TX DREQ |
| 1   | 0   | RX   | 12    | PIO1 SM0 RX DREQ |
| 1   | 0   | RX   | 13    | PIO1 SM1 RX DREQ |
| 1   | 0   | RX   | 14    | PIO1 SM2 RX DREQ |
| 1   | 0   | RX   | 15    | PIO1 SM3 RX DREQ |

## MicroPython での実装

DMA class, PIO class, StateMachine class を使って、PIO の TX_FIFO/RX_FIFO に DMA 転送を行う。

入力されたデータを Echo する PIO プログラムを実行し、TX_FIFO と RX_FIFO の両方に DMA 転送を行う例を示す。

```python
# 送受信するデータ
tx_data = array.array("I", [0x12345678, 0x9ABCDEF0, 0xDEADBEEF, 0xFEEDFACE]) # test data
rx_data = array.array("I", [0] * len(tx_data))


# PIO プログラムの定義
@rp2.asm_pio(
    # out, sideset, shift方向, autopush等の設定がここで行える
)
def pio_echoback():
    # PIO asm本体。ここではTX_FIFOの内容をRX_FIFOにそのまま返す
    wrap_target()
    pull(block)
    out(x, 32)
    in_(x, 32)
    push(block)
    wrap()

# PIO0 State Machine 0を起動
sm0 = rp2.StateMachine(0)
sm0.init(
    prog=pio_echoback,
    freq=125_000_000,  # 125MHz
    # pin設定, 周波数設定等の設定が行える
)
sm0.active(1)


# Setup TX DMA
tx_dma0 = rp2.DMA()
tx_dma0_ctrl = tx_dma0.pack_ctrl(
    size=2,  # 0=1byte, 1=2byte, 2=4byte転送
    inc_read=True,
    inc_write=False,  # PIO0 TXF0は場所固定なのでincrementしない
    treq_sel=0,  # PIO0 SM0 TX DREQ
)
tx_dma0.config(
    read=tx_data,  # 転送元データをそのまま渡す
    write=sm0,  # state machine をそのまま渡す
    count=len(
        tx_data
    ),  # 転送byte数ではなく、pack_ctrlのsizeで指定した単位での転送数なので注意
    ctrl=tx_dma0_ctrl,
    trigger=True,  # 開始
)

# Setup RX DMA
rx_dma0 = rp2.DMA()
rx_dma0_ctrl = rx_dma0.pack_ctrl(
    size=2,  # 0=1byte, 1=2byte, 2=4byte転送
    inc_read=False,  # PIO0 RXF0は場所固定なのでincrementしない
    inc_write=True,
    treq_sel=4,  # PIO0 SM0 RX DREQ
)
rx_dma0.config(
    read=sm0,  # state machine をそのまま渡す
    write=rx_data,  # 転送先データをそのまま渡す
    count=len(
        tx_data
    ),  # 転送byte数ではなく、pack_ctrlのsizeで指定した単位での転送数なので注意
    ctrl=rx_dma0_ctrl,
    trigger=True,
)

# 転送完了待ち
while rx_dma0.active():
    pass

# 結果確認
for i in range(len(rx_data)):
    print(f"TX Data[{i}]: {tx_data[i]:#010x}, RX Data[{i}]: {rx_data[i]:#010x}")
    assert tx_data[i] == rx_data[i], (
        f"Data mismatch at index {i}: {tx_data[i]:#010x} != {rx_data[i]:#010x}"
    )

# 終了処理
sm0.active(0)
tx_dma0.close()
rx_dma0.close()
```

出力

```log
TX Data[0]: 0x12345678, RX Data[0]: 0x12345678
TX Data[1]: 0x9abcdef0, RX Data[1]: 0x9abcdef0
TX Data[2]: 0xdeadbeef, RX Data[2]: 0xdeadbeef
TX Data[3]: 0xfeedface, RX Data[3]: 0xfeedface
```

## Tips

### byteswap したい

pio program 中でエンディアンが入れ替わってしまうケースなどを想定。BSWAP option があり、これは DMA の設定で行える

```python
# Setup RX DMA
rx_dma0 = rp2.DMA()
rx_dma0_ctrl = rx_dma0.pack_ctrl(
    size=2,  # 0=1byte, 1=2byte, 2=4byte転送
    inc_read=False,  # PIO0 RXF0は場所固定なのでincrementしない
    inc_write=True,
    bswap=True,  # 受信データはBig Endianなので、バイトオーダーを反転
    treq_sel=4,  # PIO0 SM0 RX DREQ
)
```

実行結果

```log
TX Data[0]: 0x12345678, RX Data[0]: 0x78563412
Traceback (most recent call last):
  File "<stdin>", line 444, in <module>
AssertionError: Data mismatch at index 0: 0x12345678 != 0x78563412
```

### 1byte ずつ転送したい

`size=0` を指定することで、1byte ずつ転送できる。転送カウント数に注意

```python
# Setup TX DMA
tx_dma0 = rp2.DMA()
tx_dma0_ctrl = tx_dma0.pack_ctrl(
    size=0,  # 0=1byte, 1=2byte, 2=4byte転送
    inc_read=True,
    inc_write=False,  # PIO0 TXF0は場所固定なのでincrementしない
    treq_sel=0,  # PIO0 SM0 TX DREQ
)
tx_dma0.config(
    read=tx_data,  # 転送元データをそのまま渡す
    write=sm0,  # state machine をそのまま渡す
    count=len(tx_data) * 4,  # 転送byte数ではなく、pack_ctrlのsizeで指定した単位での転送数なので注意
    ctrl=tx_dma0_ctrl,
    trigger=True,  # 開始
)

# RX側も同様に修正
```

結果

```log
TX Data[0]: 0x12345678, RX Data[0]: 0x12345678
TX Data[1]: 0x9abcdef0, RX Data[1]: 0x9abcdef0
TX Data[2]: 0xdeadbeef, RX Data[2]: 0xdeadbeef
TX Data[3]: 0xfeedface, RX Data[3]: 0xfeedface
```

### sniff したい、chain したい、...

`DMA.pack_ctrl()` や園周辺を読むと概ねやりたいことは記載があるはず。先に示した例を下に改造・改良していくことを推奨。

[MicroPython library - rp2.DMA](https://micropython-docs-ja.readthedocs.io/ja/latest/library/rp2.DMA.html#rp2.DMA.pack_ctrl)

## 終わりに

`DMA.config` の `read`, `write` に Peripheral が来るケースでの指定がいまいち読み取りづらかったので書き残した。

## Reference

- [MicroPython library - rp2](https://micropython-docs-ja.readthedocs.io/ja/latest/library/rp2.html)
- [RP2040 Datasheet](https://datasheets.raspberrypi.com/rp2040/rp2040-datasheet.pdf)
