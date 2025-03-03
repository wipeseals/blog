+++
title = "LiteX を Ubuntu 24.04 on WSL2 で使用し、 Demo Applicationを動作させる"
date = 2025-02-24
[taxonomies]
tags = ["tech", "litex", "fpga", "Linux", "Embedded", "riscv"]
+++

LiteX を Windows 環境で使用したく、試していたときの備忘録。以下の通り無事 demo app の動作確認までたどり着けたので書き残す。

![movie gif](/2025/setup-litex/vexriscv-donut.mp4.gif)

---

## 手順まとめ

時系列順に実行した内容のまとめを先に記載する。細かいトラブルシューティングは後半に記載。

### 環境構築

基本手順は[公式ドキュメントの Quick Start Guide](https://github.com/enjoy-digital/litex?tab=readme-ov-file#quick-start-guide)に従い、WSL2 上にインストールした Ubuntu 24.04 で行う。
要点は以下。

- virtualenv を作成し、Python 仮想環境中で実行する
- virtualenv 中で user install を実行するとエラーになるため、 `--user` オプションを使用しない
- litex の repo ごと取得した状態で `litex_setup.py` を実行しない

```sh
# verilator導入
$ sudo apt install verilator

# litexの取得
$ wget https://raw.githubusercontent.com/enjoy-digital/litex/master/litex_setup.py
$ chmod +x litex_setup.py

# venv導入
$ sudo apt install python3-venv
# 仮想環境作成 + activate
$ python3 -m venv .venv
$ source .venv/bin/activate

# litex_setup.py の実行
$ python3 ./litex_setup.py --init --install --config=full

# RSC-V toolchain の導入 (binutils-riscv* 等が導入されるので管理者権限必要)
$ pip3 install meson ninja
$ sudo ./litex_setup.py --gcc=riscv
```

### 実機無しでのテスト

ハマりポイントは特になし。verilator があれば動くはず。

<!-- more -->

```bash
# sim に必要なPackage導入
$ sudo apt install libevent-dev libjson-c-dev verilator
$ litex_sim --cpu-type=vexriscv

...
[ethernet] loaded (0x55b1423482f0)
[jtagremote] loaded (0x55b1423482f0)
[spdeeprom] loaded (addr = 0x0)
[serial2tcp] loaded (0x55b1423482f0)
[gmii_ethernet] loaded (0x55b1423482f0)
[xgmii_ethernet] loaded (0x55b1423482f0)
[serial2console] loaded (0x55b1423482f0)
[clocker] loaded
[clocker] sys_clk: freq_hz=1000000, phase_deg=0

        __   _ __      _  __
       / /  (_) /____ | |/_/
      / /__/ / __/ -_)>  <
     /____/_/\__/\__/_/|_|
   Build your hardware, easily!

 (c) Copyright 2012-2024 Enjoy-Digital
 (c) Copyright 2007-2015 M-Labs

 BIOS built on Feb 24 2025 13:45:02
 BIOS CRC passed (1cf8ec0e)

 LiteX git sha1: dd54d77db

--=============== SoC ==================--
CPU:            VexRiscv @ 1MHz
BUS:            wishbone 32-bit @ 4GiB
CSR:            32-bit data
ROM:            128.0KiB
SRAM:           8.0KiB


--============== Boot ==================--
Booting from serial...
Press Q or ESC to abort boot completely.
sL5DdSMmkekro
Timeout
No boot medium found

--============= Console ================--

litex>
```

### Arty 向けにビルド

要点

- vivado 導入前に locale の設定が必要 (en_US.UTF-8 に設定)
- WSL2 であること。もしくは何らかの X11 forwading 環境があること (Vivado install 時の GUI 表示に必要)
- vivado の path を通すこと

#### Vivado 導入

Offline installer を取得し、WSL 上で展開。WSL2 は GUI の画面転送を公式サポートしているので installer の GUI はそのまま表示される。GUI 上でセットアップを進める。

```bash
$ tar -xvf /path/to/FPGAs_AdaptiveSoCs_Unified_2024.2_1113_1001.tar
FPGAs_AdaptiveSoCs_Unified_2024.2_1113_1001/payload/rdi_0006_2024.2_1113_1001.xz
FPGAs_AdaptiveSoCs_Unified_2024.2_1113_1001/payload/rdi_0380_2024.2_1113_1001.xz
FPGAs_AdaptiveSoCs_Unified_2024.2_1113_1001/payload/rdi_0035_2024.2_1113_1001.xz
FPGAs_AdaptiveSoCs_Unified_2024.2_1113_1001/payload/sdnet_0001_2024.2_1113_1001.xz
FPGAs_AdaptiveSoCs_Unified_2024.2_1113_1001/payload/rdi_0729_2024.2_1113_1001.xz
...

# locale の設定をしておく
$ sudo locale-gen "en_US.UTF-8"
$ sudo update-locale LANG=en_US.UTF-8

# setup dir に権限必要なケースあるので sudo
$ cd FPGAs_AdaptiveSoCs_Unified_2024.2_1113_1001
$ sudo ./xsetup

# 以後 GUI Setup

# bashrc にて環境変数設定
$ echo 'source /tools/Xilinx/Vivado/2024.2/settings64.sh' >> ~/.bashrc

```

#### 論理合成

vivado に path が通っていれば、Vivado を使う FPGA の合成が通るはず。

`litex-boards/litex_boards/targets/` にボード定義があるのでこれの help を見てみると、build,load,flash など合成及び書き込みに関する情報と、ボード上の HW (usb/ethernet/....) の有効無効切り替え、CPU の設定等々がある。
WSL 上からの直接書き込みは hw_server を Host 側で立てればいけるはずだが、今回の主目的は LiteX 立ち上げなので、 Windows 側の Vivado で書き込むことにする。

```bash
$ python litex-boards/litex_boards/targets/digilent_arty.py --build
...
Creating config memory files...
Creating bitstream load up from address 0x00000000
Loading bitfile digilent_arty.bit
Writing file ./digilent_arty.bin
Writing log file ./digilent_arty.prm
===================================
Configuration Memory information
===================================
File Format        BIN
Interface          SPIX4
Size               16M
Start Address      0x00000000
End Address        0x00FFFFFF

Addr1         Addr2         Date                    File(s)
0x00000000    0x0021728B    Feb 24 16:20:10 2025    digilent_arty.bit
0 Infos, 0 Warnings, 0 Critical Warnings and 0 Errors encountered.
write_cfgmem completed successfully
# quit
INFO: [Common 17-206] Exiting Vivado at Mon Feb 24 16:20:11 2025...
```

#### 書き込み (bitstream)

`\build\digilent_arty\gateware` に Vivado の Project 一式があるので、Windows 側の Vivado Hardware Manager で `digilent_arty.bin` を書き込む。

![screenshot](/2025/setup-litex/config-bitstream.png)

#### 書き込み (SPI Flash)

同ディレクトリに `digilent_arty.bin` があるので、これを SPI Flash に書き込む。
回路図より搭載されている SPI Flash は N25Q128A13ESF40 なので、Add Configuration Memory Device でこれ (と同じ Cmd Set を持っていそうなデバイス) を指定。

![screenshot](/2025/setup-litex/config-spiflash.png)

これで FPGA の電源を入れ直しても、LiteX が起動するようになるはず。

### サンプルアプリのビルド・動作確認

自前で RISC-V toolchain を用意しビルドしても良かったが、切り分けが面倒になりそうなのでまずは Demo Application を動作させる。。
<https://github.com/enjoy-digital/litex/tree/master/litex/soc/software/demo#readme> の手順に従い `demo.bin` をビルドする。

```bash
$ itex_bare_metal_demo --build-path=build/digilent_arty
```

- <https://github.com/cr1901/teraterm-litex/releases> から install
- teraterm の転送オプションに追加された `litex` を使用し、`demo.bin` を 0x40000000 に転送する設定 + Acitve=true に設定
  - ![screenshot](/2025/setup-litex/teraterm-litex-2.png)
- VexRiscv を再起動するか、Terminal 中で `serialboot` を入力し、 `L5DdSMmkekro` が表示されたら転送が開始される
- 無事に demo app が起動したら、 `donut` を入力して無事動作確認 OK
  - ![screenshot](/2025/setup-litex/teraterm-litex-3.png)

---

## Troubleshooting

戦術の内容を実行した際の経緯等々。

### migen 導入時の `python -m pip install` が通らない

PEP 668 に従い、システムパッケージを破壊するリスクがあるため弾かれている。

```sh
$ python3 ./litex_setup.py --init --install --user --config=full
...

[   0.451] Installing migen Git repository...

error: externally-managed-environment

× This environment is externally managed
╰─> To install Python packages system-wide, try apt install
    python3-xyz, where xyz is the package you are trying to
    install.

    If you wish to install a non-Debian-packaged Python package,
    create a virtual environment using python3 -m venv path/to/venv.
    Then use path/to/venv/bin/python and path/to/venv/bin/pip. Make
    sure you have python3-full installed.

    If you wish to install a non-Debian packaged Python application,
    it may be easiest to use pipx install xyz, which will manage a
    virtual environment for you. Make sure you have pipx installed.

    See /usr/share/doc/python3.12/README.venv for more information.

note: If you believe this is a mistake, please contact your Python installation or OS distribution provider. You can override this, at the risk of breaking your Python installation or OS, by passing --break-system-packages.
hint: See PEP 668 for the detailed specification.
Traceback (most recent call last):
  File "/mnt/e/repos/litex/./litex_setup.py", line 497, in <module>
    main()
  File "/mnt/e/repos/litex/./litex_setup.py", line 477, in main
    litex_setup_install_repos(config=args.config, user_mode=args.user)
  File "/mnt/e/repos/litex/./litex_setup.py", line 290, in litex_setup_install_repos
    subprocess.check_call("\"{python3}\" -m pip install {editable} . {options}".format(
  File "/usr/lib/python3.12/subprocess.py", line 413, in check_call
    raise CalledProcessError(retcode, cmd)
```

エラー記載のとおり仮想環境で実行する

```sh
# venv導入
$ sudo apt install python3-venv
# 仮想環境作成 + activate
$ python3 -m venv .venv
$ source .venv/bin/activate
# 再実行
$ python3 ./litex_setup.py --init --install --user --config=full
```

### 上記対応した後に `--user` の pip install が通らない

```sh
[   0.065] Installing migen Git repository...
ERROR: Can not perform a '--user' install. User site-packages are not visible in this virtualenv.
Traceback (most recent call last):
  File "/mnt/e/repos/litex/./litex_setup.py", line 497, in <module>
    main()
  File "/mnt/e/repos/litex/./litex_setup.py", line 477, in main
    litex_setup_install_repos(config=args.config, user_mode=args.user)
  File "/mnt/e/repos/litex/./litex_setup.py", line 290, in litex_setup_install_repos
    subprocess.check_call("\"{python3}\" -m pip install {editable} . {options}".format(
  File "/usr/lib/python3.12/subprocess.py", line 413, in check_call
    raise CalledProcessError(retcode, cmd)
subprocess.CalledProcessError: Command '"/mnt/e/repos/litex/.venv/bin/python3" -m pip install  . --user' returned non-zero exit status 1.
```

pip の install 先は global, user, virtualenv の 3 段階ある。
`litex_setup.py` の `--user` option を外せばよいがこれは global になるので避けたい。

今回のエラーは user install を virtualenv 上で実行しようとしているために起きている。

ややこしいので表にまとめる。現在遭遇しているケースが 4 に該当する。

| #   | 仮想環境 | `--user` | install 先                                    | location                                  |
| --- | -------- | -------- | --------------------------------------------- | ----------------------------------------- |
| 1   | なし     | なし     | System                                        | `/usr/local/lib/python3.12/dist-packages` |
| 2   | なし     | あり     | User                                          | `~/.local/lib/python3.12/site-packages`   |
| 3   | あり     | なし     | Virtualenv                                    | `.venv/lib/python3.12/site-packages`      |
| 4   | あり     | あり     | User (w/ `include-system-site-packages=true`) | `~/.local/lib/python3.12/site-packages`   |

`.venv/pyvenv.cfg` に `include-system-site-packages = false` がある場合は 4 はエラーになるが、 `true` に設定すると User install が実行される。User install で構わないのであればこれも解決策ではある。 (venv を切った意味は無さそう)

今回は、 3. を選択することにし、 `litex_setup.py` から `--user` を外して実行する。

```bash
$ source .venv/bin/activate
$ python3 ./litex_setup.py --init --install  --user --config=full
```

### 上位ディレクトリに subrepo をばらまいているように見える

`litex_setup.py` を実行すると、 通常同一ディレクトリに他のリポジトリが展開される。
litex の git repo ごと取得して実行していたが、実行箇所が git repo の場合は上位ディレクトリに展開されるようだった。

Quick Start Guide 通り、 `litex_setup.py` のみを取得して実行するように修正。

### vivado セットアップ時・実行時に locale のエラー

以下表示でセットアップ失敗。vivado 自体はあるので実行できるが同様にエラーになる。

```bash
######## Execution of Pre/Post Installation Tasks Failed ########
Warning: AMD software was installed successfully, but an unexpected status was returned from the following post installation task(s) /tools/Xilinx/Vivado/2024.2/bin/rdiArgs.sh: line 37: warning: setlocale: LC_ALL: cannot change locale (en_US.UTF-8): No such file or directory /bin/bash: warning: setlocale: LC_ALL: cannot change locale (en_US.UTF-8) terminate called after throwing an instance of 'std::runtime_error' what(): locale::facet::_S_create_c_locale name not valid /tools/Xilinx/Vivado/2024.2/bin/rdiArgs.sh: line 421: 74809 Aborted "$RDI_PROG" "$@" /tools/Xilinx/Vivado/2024.2/bin/rdiArgs.sh: line 37: warning: setlocale: LC_ALL: cannot change locale (en_US.UTF-8): No such file or directory /bin/bash: warning: setlocale: LC_ALL: cannot change locale (en_US.UTF-8) terminate called after throwing an instance of 'std::runtime_error' what(): locale::facet::_S_create_c_locale name not valid /tools/Xilinx/Vivado/2024.2/bin/rdiArgs.sh: line 421: 74902 Aborted "$RDI_PROG" "$@"
```

en_US.UTF-8 の locale を設定する。

参考: <https://adaptivesupport.amd.com/s/question/0D54U00006FYojlSAD/vivado-20222-on-ubuntu-with-error-lcall-cannot-change-locale-enusutf8?language=ja>

```bash
$ sudo locale-gen "en_US.UTF-8"
$ sudo update-locale LANG=en_US.UTF-8
```
