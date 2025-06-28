+++
title = "My Works"
path = "work"
template = "page.html"
+++

## nandio.pio

A project that accelerates NAND Flash communication using the Raspberry Pi Pico's PIO (Programmable I/O).

![img](https://github.com/wipeseals/nandio.pio/raw/master/misc/PioNandCommander-ProgramPage-Core125MHz-Pio125MHz.png)

- Online Simulation Report: [https://wipeseals.github.io/nandio.pio/](https://wipeseals.github.io/nandio.pio/)
- GitHub: [https://github.com/wipeseals/nandio.pio](https://github.com/wipeseals/nandio.pio)

---

## UdonKnucklebones

This is a playable implementation of the dice game "Knucklebones" designed for the social VR platform, VRChat.

![img](https://github.com/wipeseals/UdonKnucklebones/raw/main/Docs~/screenshot/asset-preview.png)

- GitHub: [https://github.com/wipeseals/UdonKnucklebones](https://github.com/wipeseals/UdonKnucklebones)
- BOOTH: [https://wipeseals.booth.pm/items/6278798](https://wipeseals.booth.pm/items/6278798)

---

## litex-multibinary-combiner

A utility designed to merge multiple LiteX binaries into a single, combined binary file.

```bash
$ python main.py /path/to/boot.json
[17:52:18] INFO     Boot JSON Path: D:\Data\Downloads\linux_2022_03_23\boot.json                                                         main.py:68
           INFO     Output File: D:\Data\Downloads\linux_2022_03_23\Combined.bin                                                         main.py:69
           INFO     Initial Value: 0                                                                                                     main.py:70
           INFO     Start Address: 0x0000000040000000                                                                                    main.py:71
           INFO     End Address: 0x000000004139b400                                                                                      main.py:72
           INFO     Total Size: 20558848 bytes                                                                                           main.py:73
           INFO     0x0000000040000000 - 0x000000004072ebcc: 7531468 bytes  Image                                                        main.py:89
           INFO     0x000000004072ebcc - 0x0000000040ef0000: 8131636 bytes  Fill with 0x00                                               main.py:83
           INFO     0x0000000040ef0000 - 0x0000000040ef07cf: 1999 bytes     rv32.dtb                                                     main.py:89
           INFO     0x0000000040ef07cf - 0x0000000040f00000: 63537 bytes    Fill with 0x00                                               main.py:83
           INFO     0x0000000040f00000 - 0x0000000040f0d188: 53640 bytes    opensbi.bin                                                  main.py:89
           INFO     0x0000000040f0d188 - 0x0000000041000000: 994936 bytes   Fill with 0x00                                               main.py:83
           INFO     0x0000000041000000 - 0x000000004139b400: 3781632 bytes  rootfs.cpio                                                  main.py:89
           INFO     Done. Output: D:\Data\Downloads\linux_2022_03_23\Combined.bin                                                        main.py:95
```

- GitHub: [https://github.com/wipeseals/litex-multibinary-combiner](https://github.com/wipeseals/litex-multibinary-combiner)

---

## riscv-inst-viewer

A tool that allows you to search and decode the RISC-V instruction set.

input

```text
0x0010051b
```

output

```text
ADDIW x10, x0, 1
Instruction: ADDIW (Add Immediate Word)
Format: I-Type (I / RV64)
Opcode: 0011011
rd: x10 (val: 10)
funct3: 000
rs1: x0 (val: 0)
rs2: x1 (val: 1)
funct7: 0000000
Immediate: 1 (0x1)
```

- Viewer: [https://wipeseals.github.io/riscv-inst-viewer/](https://wipeseals.github.io/riscv-inst-viewer/)
- GitHub: [https://github.com/wipeseals/riscv-inst-viewer](https://github.com/wipeseals/riscv-inst-viewer)

---

## asciidrom

A converter that transforms Wavedrom JSON timing diagrams into ASCII art suitable for embedding in source code comments.

input

```js
{ signal: [
  { name: "clk", wave: "p..P..p..P..p" },
  { name: "req", wave: "0.1..0....10." },
  { name: "bus", wave: "x.22.333.4.x.5x", data: ["A", "B", "C", "D"] },
  {}, // spacer
  ["Group Name",
    { name: "ack", wave: "0..1..0..1.0." },
    { name: "data", wave: "x..2..x..3.x.", data: ["d1", "d2"] }
  ]
],
}
```

output

```text
clk        | _/¯¯\__/¯_/¯¯\__/¯_/¯¯\__/¯_/¯¯\__/¯_/¯¯\__/¯
req        | _______/¯¯¯¯¯¯¯¯\______________/¯¯\__________
bus        | xxxxxx=A==B=====C==D===========xxxxxx==xxx
           |
Group Name
  ack      | __________/¯¯¯¯¯¯¯¯\________/¯¯¯¯¯\__________
  data     | xxxxxxxxx=d=======xxxxxxxxx=d====xxxxxxxxxxxx
```

- Viewer: [https://wipeseals.github.io/asciidrom/](https://wipeseals.github.io/asciidrom/)
- GitHub: [https://github.com/wipeseals/asciidrom](https://github.com/wipeseals/asciidrom)

---

## nvme-over-pcie-bar0-viewer

A parser and validator for binary dumps of NVMe (Non-Volatile Memory Express) Controller Registers, specifically over PCIe BAR0.

input

```text
00000000: ff ff 03 3c 30 00 00 00 00 04 01 00 00 00 00 00
00000010: 00 00 00 00 01 40 46 00 00 00 00 00 09 00 00 00
00000020: 00 00 00 00 1f 00 1f 00 00 c0 a4 ff 00 00 00 00
00000030: 00 d0 a4 ff 00 00 00 00 00 00 00 00 00 00 00 00
(snipped)
```

output

```text
Offset  Register   Raw Bytes (LE)   Parsed & Validated Details
0x00    CAP        ff ff 03 3c 30 00 00 00

Controller Capabilities
Value: 0x000000303C03FFFF

Field        Bit(s)     Hex     Parsed Value & Validation
MQES         15:0       0xFFFF  65536 entries
CQR          16         0x01    Yes
AMS (WRR)    17         0x01    Weighted Round Robin: Supported
AMS (VS)     18         0x00    Vendor Specific: Not Supported
Reserved0    23:19      -
TO           31:24      0x3C    30000 ms
DSTRD        35:32      0x00    4 bytes
NSSRS        36         0x01    Yes
CSS (NVM)    37         0x01    NVM Command Set: Supported
Reserved1    44:38      -
BPS          45         0x00    No
Reserved2    47:46      -
MPSMIN       51:48      0x00    4096 B
MPSMAX       55:52      0x00    4096 B
Reserved3    63:56      -
(snipped)
```

- Viewer: [https://wipeseals.github.io/nvme-over-pcie-bar0-viewer/](https://wipeseals.github.io/nvme-over-pcie-bar0-viewer)
- GitHub: [https://github.com/wipeseals/nvme-over-pcie-bar0-viewer](https://github.com/wipeseals/nvme-over-pcie-bar0-viewer)

---

## dot-illust-converter

An image converter that transforms pictures into a retro, pixel-art game style.

- Online: [https://wipeseals.github.io/dot-illust-converter/](https://wipeseals.github.io/dot-illust-converter/)
- GitHub: [https://github.com/wipeseals/dot-illust-converter](https://github.com/wipeseals/dot-illust-converter)
