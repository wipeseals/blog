+++
title = "Raspberry PI PIO で NANDアクセスを高速化する"
date = 2024-10-30
[taxonomies]
tags = ["tech", "slide", "pio", "Embedded", "raspberrypi", "Driver", "NAND"]
+++

登壇する機会があったのでその際の資料。RP2040 に搭載されている Programmable IO を用いて、Parallel Command I/F である NAND IC へのアクセスを高速化検討した話。

[rpi-pio-nand.pdf](/2024/rpi-pio-nand/rpi-pio-nand.pdf)

資料は marp を使用して作成したので、以後は元になった資料の markdown 移植版

## 背景

最近 SSD 自作キット JISC-SSD で ~~時折放置してますが~~ 遊んでいます。
途中まで C++で書いていましたが、気まぐれで Rust 再実装中...

RPi Pico: RP2040 Cortex-M0+ DualCore @133MHz
NAND Flash: KIOXIA TC58NVG0S3HTA00 1Gbit SLC ECC なし

| 外観                                                   | 回路図抜粋                                             | RP2040 Block Diagram                                  |
| ------------------------------------------------------ | ------------------------------------------------------ | ----------------------------------------------------- |
| ![width:400px](/2024/rpi-pio-nand/sashie01-scaled.jpg) | ![width:700px](/2024/rpi-pio-nand/jisc-ssd-schema.png) | ![width:600px](/2024/rpi-pio-nand/rp2040-diagram.png) |
|                                                        | GPIO で NAND と直結                                    |                                                       |

<!-- more -->

---

## 課題

NAND IC へのアクセスをもう少しいい感じにしたい

GPIO 直叩きでの通信は遅い (当然と言えばそう)
SIO (Single Cycle IO) もあるが、転送中 CPU Resouce 占有してしまう
**PIO (Programmable IO) という小規模なシーケンサのような HW が乗っている**

| SIO                                                | PIO Overview                                      |
| -------------------------------------------------- | ------------------------------------------------- |
| ![width:1000px](/2024/rpi-pio-nand/rp2040-sio.png) | ![width:600px](/2024/rpi-pio-nand/rp2040-pio.png) |
| bus fabric 経由せず GPIO 叩ける                    | State Machine 4 機                                |

---

## PIO 概要

- 動作周波数はデフォルトで 125MHz, uCode は 32 命令まで
- 命令セットはデータ入出力+入出力時の bitshift、分岐命令 あたり
- GPIO の Read/Write/PinDir 変更、FIFO 経由で CPU/DMA とやりとりできる
- 命令実行と同時に固定サイクル遅延 (Delay)、もしくは特定ピンの H/L を変更できる (Sideset)
  - この 2 機能に割り当てられる bit は合わせて 5bit まで

| 命令セット                                                 | Shift Register                                                                                                                                                       |
| ---------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ![height:400px](/2024/rpi-pio-nand/rp2040-pio-opcodes.png) | TX FIFO -> Output Shift Reg ![height:200px](/2024/rpi-pio-nand/rp2040-pio-osr.png) Input Shift Reg -> RX FIFO ![height:120px](/2024/rpi-pio-nand/rp2040-pio-isr.png) |
| Delay/sideset の bit 割り振りは起動前に設定                |                                                                                                                                                                      |

---

## NAND IC との通信

GPIO 複数ピンによるコマンド入力 + 双方向データ転送が必要

| Pin      | Function                                     |
| -------- | -------------------------------------------- |
| /CS      | Chip Enable                                  |
| /WE, /RE | Write/Read Enable。転送クロック相当          |
| ALE, CLE | Address/Command Latch Enable。入力データ区別 |
| RY//BY   | Ready/Busy: Busy なら Low                    |
| IO[7:0]  | データ線。双方向                             |

| Logic & Command Table                                | Read Operation                                                        |
| ---------------------------------------------------- | --------------------------------------------------------------------- |
| ![width:750px](/2024/rpi-pio-nand/nand-cmdtable.png) | ![width:1000px](/2024/rpi-pio-nand/nand-read.png)                     |
|                                                      | Cmd In (0x00)-> Addr In -> Cmd In (0x30) -> Wait RY/BY -> Data Out... |

---

## Simulation/Debug 環境

デバッガがないため、 Jupyter 上に論理検証環境構築
adafruit_pioasm + pioemu + pandas + wavedrom

![width:1800px](/2024/rpi-pio-nand/pio-readid-debug-manual.png)

---

## 設計

### DMA で流すデータ上に独自コマンドセットを定義

CmdId を指定し、各シーケンスごとのルーチンを実装。CPU は Buffer 上にシーケンスを組む
(流すデータの順番があっていればよいので、連続領域に確保しなくても DMA を Chain で良い)

| Data[0] assign | Field         | Description                                      |
| -------------- | ------------- | ------------------------------------------------ |
| [31:28]        | CmdId         | コマンド指定                                     |
| [27:16]        | TransferCount | (AddrLatch/DataIn/DataOut のみ) 転送 byte 数指定 |
| [15:0]         | PinDirection  | GPIO の入出力設定                                |

| CmdId      | Data[1, ...]  | Description                                 |
| ---------- | ------------- | ------------------------------------------- |
| Bitbang    | PinData       | そのまま GPIO に流す                        |
| CmdLatch   | NandCmdId     | CLE 有効で Data 入力                        |
| AddrLatch  | NandAddr      | ALE 有効で Data 入力                        |
| DataOutput | -             | 指定回数 Data 出力 + GPIO Read              |
| DataInput  | TransferDatas | 指定回数 Data 入力                          |
| WaitRbb    | -             | RB/BY pin が Ready になるまでループ         |
| SetIrq     | -             | 割り込み通知（転送完了を CPU に知らせる用） |

---

| PIO Process Diagram                                      | Sequence                                                   |
| -------------------------------------------------------- | ---------------------------------------------------------- |
| ![height:900px](/2024/rpi-pio-nand/pio-cmd-overview.svg) | ![width:1100px](/2024/rpi-pio-nand/pio-read-operation.png) |

---

## 性能 (Data Input の例)

$f_{Max}$ で動かすと Data の Setup 満たさないので 83MHz (12ns) より下げて動かす必要あり。
GPIO Toggle よりはだいぶ速くなったが、AC 特性 Spec と比べるともうひと声。

![height:740px](/2024/rpi-pio-nand/pio-write-operation-timing.png)

---

## まとめ

### 所感

PIO は小規模ながら色々できて面白い
System Clock (Max 133MHz) で動くのでホビー用途なら十分速い
実機デバッグはしんどいので Sim 環境作っておいてよかった
FTL としての続きもぼちぼち実装します

### 出典

JISC-SSD(Jisaku In-Storage Computation SSD 学習ボード) - 株式会社クレイン電子
RP2040 Datasheet - Raspberry Pi Ltd
TC58NVG0S3HTA00 Datasheet - KIOXIA

---

Fin.
