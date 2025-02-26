+++
title = "Docker で riscv-tests をビルドする"
date = 2025-02-26
[taxonomies]
tags = ["tech", "Docker", "riscv"]
+++

(ビルドが長くて) 地味に時間がかかったので、今後時間を溶かさないように書き残す。

## 手順

### Dockerfile で riscv-gnu-toolchain 環境を用意

これはほぼ公式手順を Dockerfile に書き起こすだけで良い。`ARCH`, `ABI` などは好みのものに変更する。

```Dockerfile
FROM ubuntu:24.04

WORKDIR /app

# riscv-gnu-toolchainのリポジトリを取得
RUN apt update && apt install -y git
RUN git clone --recursive https://github.com/riscv/riscv-gnu-toolchain

ENV RISCV=/opt/riscv
ENV ARCH=rv32ima
ENV ABI=ilp32
ENV PATH=$RISCV/bin:$PATH
ENV TOOLCHAIN_CONFIGURE="--prefix=$RISCV --with-arch=$ARCH --with-abi=$ABI --enable-multilib --enable-qemu-system"
ENV DEBIAN_FRONTEND=noninteractive

# 必要なパッケージをインストール
RUN apt install -y \
    autoconf \
    automake \
    autotools-dev \
    curl \
    python3 \
    python3-pip \
    python3-tomli \
    libmpc-dev \
    libmpfr-dev \
    libgmp-dev \
    gawk \
    build-essential \
    bison \
    flex \
    texinfo \
    gperf \
    libtool \
    patchutils \
    bc \
    zlib1g-dev \
    libexpat-dev \
    ninja-build \
    git \
    cmake \
    libglib2.0-dev \
    libslirp-dev

# riscv-gnu-toolchainのインストール
RUN cd riscv-gnu-toolchain && \
    ./configure $TOOLCHAIN_CONFIGURE && \
    make -j$(nproc)

CMD ["bash"]
```

### 上記 Dockerfile を使って riscv-tests をビルド

multi-stage build で riscv-tests を行っても良かったが、単にビルドするだけであること、自分が実装した別ソースをビルドすることなどを考え先の Image を利用するだけに留めることにした。

docker compose を用意し、成果物 Volume のマウント並びにビルドのワンライナーを用意する。
実装してある内容は riscv-tests の README 通りだが、今回は rv32 を対象にしているので XLEN を設定していること、ISA テスト用に make 対象を isa に絞っている。

```yaml
services:
  build-riscv-tests:
    build:
      context: .
    volumes:
      - ./riscv-tests:/app/riscv-tests:rw
    command: >
      bash -c "
      cd /app/riscv-tests && 
      git clone --recursive https://github.com/riscv/riscv-tests && 
      cd riscv-tests && 
      autoconf && 
      ./configure --prefix=$RISCV/target && 
      make XLEN=32 isa
      "
```

使い方

```bash
docker compose run build-riscv-tests
```

## その他

### riscv-gnu-toolchain のその他のツールを使いたい

`docker compose run build-riscv-tests bash` で入った環境であれば、他のツールも使える。

```bash
$ docker compose run build-riscv-tests bash
root@460560b02bb5:/app# riscv32-unknown-elf-readelf -e -W riscv-tests/isa/rv32ui-p-addi
ELF Header:
  Magic:   7f 45 4c 46 01 01 01 00 00 00 00 00 00 00 00 00
  Class:                             ELF32
  Data:                              2's complement, little endian
  Version:                           1 (current)
  OS/ABI:                            UNIX - System V
  ABI Version:                       0
  Type:                              EXEC (Executable file)
  Machine:                           RISC-V
  Version:                           0x1
  Entry point address:               0x80000000
  Start of program headers:          52 (bytes into file)
  Start of section headers:          9524 (bytes into file)
  Flags:                             0x0
  Size of this header:               52 (bytes)
  Size of program headers:           32 (bytes)
  Number of program headers:         3
  Size of section headers:           40 (bytes)
  Number of section headers:         7
  Section header string table index: 6

Section Headers:
  [Nr] Name              Type            Addr     Off    Size   ES Flg Lk Inf Al
  [ 0]                   NULL            00000000 000000 000000 00      0   0  0
  [ 1] .text.init        PROGBITS        80000000 001000 00047c 00  AX  0   0 64
  [ 2] .tohost           PROGBITS        80001000 002000 000048 00  WA  0   0 64
  [ 3] .riscv.attributes RISCV_ATTRIBUTES 00000000 002048 000063 00      0   0  1
  [ 4] .symtab           SYMTAB          00000000 0020ac 0002b0 10      5  37  4
  [ 5] .strtab           STRTAB          00000000 00235c 000198 00      0   0  1
  [ 6] .shstrtab         STRTAB          00000000 0024f4 000040 00      0   0  1
Key to Flags:
  W (write), A (alloc), X (execute), M (merge), S (strings), I (info),
  L (link order), O (extra OS processing required), G (group), T (TLS),
  C (compressed), x (unknown), o (OS specific), E (exclude),
  D (mbind), p (processor specific)

Program Headers:
  Type           Offset   VirtAddr   PhysAddr   FileSiz MemSiz  Flg Align
  RISCV_ATTRIBUT 0x002048 0x00000000 0x00000000 0x00063 0x00000 R   0x1
  LOAD           0x001000 0x80000000 0x80000000 0x0047c 0x0047c R E 0x1000
  LOAD           0x002000 0x80001000 0x80001000 0x00048 0x00048 RW  0x1000

 Section to Segment mapping:
  Segment Sections...
   00     .riscv.attributes
   01     .text.init
   02     .tohost
```

### RV64/32 の選択

- Dockerfile の `ARCH` と `ABI` を変更する
- make 時の `XLEN` を変更する
  - riscv-tests のデフォルトは `XLEN=64` を使おうとしているので、rv32 向けの toolchain だと gcc などが見つからずエラーになる

### Linker script の修正

[riscv-tests/env が更に submodule になっており](https://github.com/riscv/riscv-test-env)、ここを修正することで対応できる。
例えば `p/link.ld` は以下の具合

```ld
OUTPUT_ARCH( "riscv" )
ENTRY(_start)

SECTIONS
{
  . = 0x80000000;
  .text.init : { *(.text.init) }
  . = ALIGN(0x1000);
  .tohost : { *(.tohost) }
  . = ALIGN(0x1000);
  .text : { *(.text) }
  . = ALIGN(0x1000);
  .data : { *(.data) }
  .bss : { *(.bss) }
  _end = .;
}
```

この他にも `riscv_test.h` にテストの各マクロの定義があるので、動作確認向けに変更することもできる。
例えば Pass/Fail は、Pass の場合

```c
//-----------------------------------------------------------------------
// Pass/Fail Macro
//-----------------------------------------------------------------------

#define RVTEST_PASS                                                     \
        fence;                                                          \
        li TESTNUM, 1;                                                  \
        li a7, 93;                                                      \
        li a0, 0;                                                       \
        ecall

#define TESTNUM gp
#define RVTEST_FAIL                                                     \
        fence;                                                          \
1:      beqz TESTNUM, 1b;                                               \
        sll TESTNUM, TESTNUM, 1;                                        \
        or TESTNUM, TESTNUM, 1;                                         \
        li a7, 93;                                                      \
        addi a0, TESTNUM, 0;                                            \
        ecall
```

### disassemble が見たい

付属ツールで disasm 出力しても良かったが、make 時に一緒に生成されていた。 `<elf binary>.dump` がそれにあたる。

`rv32ui-p-addi.dump`

```asm
rv32ui-p-addi:     file format elf32-littleriscv


Disassembly of section .text.init:

80000000 <_start>:
80000000:	0500006f          	j	80000050 <reset_vector>

80000004 <trap_vector>:
80000004:	34202f73          	csrr	t5,mcause
80000008:	00800f93          	li	t6,8
8000000c:	03ff0863          	beq	t5,t6,8000003c <write_tohost>

...


800003fc <test_25>:
800003fc:	01900193          	li	gp,25
80000400:	02100093          	li	ra,33
80000404:	03208013          	addi	zero,ra,50
80000408:	00000393          	li	t2,0
8000040c:	00701463          	bne	zero,t2,80000414 <fail>
80000410:	02301063          	bne	zero,gp,80000430 <pass>

80000414 <fail>:
80000414:	0ff0000f          	fence
80000418:	00018063          	beqz	gp,80000418 <fail+0x4>
8000041c:	00119193          	slli	gp,gp,0x1
80000420:	0011e193          	ori	gp,gp,1
80000424:	05d00893          	li	a7,93
80000428:	00018513          	mv	a0,gp
8000042c:	00000073          	ecall

80000430 <pass>:
80000430:	0ff0000f          	fence
80000434:	00100193          	li	gp,1
80000438:	05d00893          	li	a7,93
8000043c:	00000513          	li	a0,0
80000440:	00000073          	ecall
80000444:	c0001073          	unimp
80000448:	0000                	.insn	2, 0x
8000044a:	0000                	.insn	2, 0x
8000044c:	0000                	.insn	2, 0x
8000044e:	0000                	.insn	2, 0x
80000450:	0000                	.insn	2, 0x
80000452:	0000                	.insn	2, 0x
80000454:	0000                	.insn	2, 0x
80000456:	0000                	.insn	2, 0x
80000458:	0000                	.insn	2, 0x
8000045a:	0000                	.insn	2, 0x
8000045c:	0000                	.insn	2, 0x
8000045e:	0000                	.insn	2, 0x
80000460:	0000                	.insn	2, 0x
80000462:	0000                	.insn	2, 0x
80000464:	0000                	.insn	2, 0x
80000466:	0000                	.insn	2, 0x
80000468:	0000                	.insn	2, 0x
8000046a:	0000                	.insn	2, 0x
8000046c:	0000                	.insn	2, 0x
8000046e:	0000                	.insn	2, 0x
80000470:	0000                	.insn	2, 0x
80000472:	0000                	.insn	2, 0x
80000474:	0000                	.insn	2, 0x
80000476:	0000                	.insn	2, 0x
80000478:	0000                	.insn	2, 0x
8000047a:	0000                	.insn	2, 0x
```
