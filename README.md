# Guide for BCOMP

### Disclaimer
**THIS IT NOT COMPREHENSIVE DESCRIPTION! USE OFFICIAL [METHODICAL](https://se.ifmo.ru/documents/10180/38002/Методические+указания+к+выполнению+лабораторных+работ+и+рубежного+контроля+БЭВМ+2019+bcomp-ng.pdf/d5a1be02-ad3f-4c43-8032-a2a04d6db12e) FOR MORE INFO!**

### This guide in russian: https://github.com/Zerumi/OPD-guide-RU-

### THE biggest mistake in your life
In BCOMP, plain numbers like `1`, `15` are treated as ***decimal*** not as ***hex***!
Be careful and don't miss the `0x` prefix! Actual hex number would be: `0x15`
Example: 
```asm
in 15   ; this is DECIMAL! Reads literally 
        ; from fifteenth external register (actual address is 0xF)
        ; that is from External Device 5 - Text Printer

; vs
in 0x15 ; reads from twenty first (21 in decimal) external register
        ; that is from External Device 7 - 7 segment indicator
        ; from status register

```

## Commands
- **[LOOP](loop.md)**
- **[Micro-commands](microcode.md)**
- **[IO (Input Output)](io.md)**

## Reports examples
 You can found examples of reports which we at least partially accepted by teacher in [`reports`](/reports/) folders.

## Table of content
- [Guide for BCOMP](#guide-for-bcomp)
    - [Disclaimer](#disclaimer)
    - [This guide in russian: https://github.com/Zerumi/OPD-guide-RU-](#this-guide-in-russian-httpsgithubcomzerumiopd-guide-ru-)
    - [THE biggest mistake in your life](#the-biggest-mistake-in-your-life)
  - [Commands](#commands)
  - [Reports examples](#reports-examples)
  - [Table of content](#table-of-content)
  - [Modes](#modes)
  - [Assembly](#assembly)
  - [Load `asm` file into bcomp](#load-asm-file-into-bcomp)
  - [CLI](#cli)
  - [Trace](#trace)
  - [Addressing](#addressing)
    - [Notes on *Indirect relative*](#notes-on-indirect-relative)
  - [Command execution stages](#command-execution-stages)

## Modes

BCOMP can be launched in next modes:
 - `gui` -- default mode (graphical interface)
 - `cli` -- command line interface
 - `decoder` -- dump default micro-program (microcode) into terminal
 - `nightmare` -- literally nightmare, don't launch or you will get mental trauma
 - `dual` -- launches bcomp in both `gui` and `cli` modes simultaneously (**recommended for [tracing](#trace)**)

To launch bcomp in certain mode specify flag <code>-Dmode=<em>mode_name</em></code> and replace `mode_name` with one of the above. Example:
```
java -jar -Dmode=dual bcomp-ng.jar
```

## Assembly
Assembly code can be written both in foreign text editors (like vscode or vim) and in `cli` mode.

In general, assembly is written alike below:
```
[LABEL: ] COMMAND_MNEMONIC ARGUMENT
```
*Note:* square brackets means that this part is optional. To see full syntax look at **page 51** in [methodical](https://se.ifmo.ru/documents/10180/38002/Методические+указания+к+выполнению+лабораторных+работ+и+рубежного+контроля+БЭВМ+2019+bcomp-ng.pdf/d5a1be02-ad3f-4c43-8032-a2a04d6db12e)

Labels are useful to refer to some location in memory. Read more in [addressing](#addressing) section.

Numbers can be written as decimal (e.g. `15`) or hexadecimal (e.g. `0xF`).

Here some *special* commands that might come in handy:
| Command                   | Description                                                                                                                                                                        |
| ------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `org ADDRESS`             | guides the compiler to place next value <br> (be it a raw value or a command) into cell <br> with address `ADDRESS`. Subsequent commands will be placed after `ADDRESS` one by one |
| `word 0x0000,0xffa4,0xfa` | Places specified value into memory as is. <br> `?` is equal to `0x0000` in this context <br> `0x15` is equal to `0x0015`                                                           |

*Note:* you can use any case of letters you desire: `0xfA51`, `0xFF`, `0xac` are all valid.

These special commands are not represented in `bcomp` memory. They are intended to assist you write assembly code

There is special label `START`. It tells bcomp from where to start execution. Address, to which `START` refers, is placed into `IP`. This means, the program can be executed right off start of bcomp.


Here is simple example (For more advanced one look [here](example.asm)):

```asm
org 0x04f
VAR1: word 0x45a9
START: add 0xf
sub VAR1
```
Here `0x45a9` will be placed in memory as is at address `0x04f`; `add 0xf` at `0x050` and so on. Also `0x050` (the address with special label `START`) is loaded into `IP`. Thus you don't have to manually setup this.

## Load `asm` file into bcomp
BCOMP has additional parameter to load file with code: <code>-Dcode=<em>file</em></code> where `file` is any valid path to existing file: `foobar.asm`, `./keklol`, `/home/foo/kek` are all valid.

*Note:* `.asm` extension is optional and is **not required**. File can have any name that your OS accepts.

So to load some code and trace it you would type something like:
```
java -jar -Dmode=dual -Dcode=main.asm bcomp-ng.jar
```

## CLI
CLI is pretty cool. Finally you can input hexadecimal numbers. Seriously, just type it:
```
ffae
``` 
This will set `Input Register` to the value you wrote.

Then, you can type:
```
a
```
and value of `Input Register` will be loaded into `IP` register (analogue to press of `Enter Address` button in `gui` mode).
For more commands see `help`.

You can also write assembly right in BCOMP:
```
asm
Введите текст программы. Для окончания введите END
add 0xff
sub (0xf1)
end
```

## Trace
Launch BCOMP in `dual` mode and load your `.asm` file like described in [this](#load-asm-file-into-bcomp) section.
To see nice trace table in your terminal press `continue` button in GUI. You will see how neat lines appear in your terminal.

After program has been executed completely:
1. Copy trace table from terminal. 
1. Paste it into raw text file e.g. `text.txt`. 
1. Replace all spaces (` `) with comma (`,`). 
1. Rename file to something like <code>text<strong>.csv</strong></code>.
1. Open the file with Libreoffice Calc or another sheet processing program
1. Copy table from Calc to Libreoffice Writer or another program you are writing report in
1. Format header of the table

## Addressing

When you see something like this <code>2<strong>E</strong>F5</code> the second letter (`E` here) is responsible for addressing mode. Look at the table below (`L` stand for `Label`)

| Hex code | Name                   | Notation                     | Example                            | Description                                                                                                                                                                                                                                                                                                                                                                                          |
| -------- | ---------------------- | ---------------------------- | ---------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 0x0-0x7  | Absolute               | `add $L` <br> `add ADDR`     | `add $VAR1` <br> `add 0xf`         | Add number from memory cell with address `0xf` or from `$VAR1` label                                                                                                                                                                                                                                                                                                                                 |
| 0xE      | Direct relative        | `add L`                      | `add VAR1` <br> `4EFE`             | **Only labels are supported!** `IP + 1 + OFFSET`. Notice, that `IP` point to *next* command. So that is where `+ 1` comes from. Offset can be *positive* and *negative*. Something like `0x80`, `0xfe` is **negative**. Let's assume that `4EFE` (`add`) have address `0x010`. Therefore it points to address right before `4EFE` (`0xFE` is *negative* = `-2`) i.e. `0x009`. `4EFF` point to itself |
| 0x8      | Indirect relative      | `add (L)`                    | `add (VAR1)`                       | Works like pointers in C/C++. It's like saying: *Hey, look in this box. Here you find paper which tells you where exactly the thing is.* `add` --(Direct relative)--> `VAR1` --(Absolute)--> `value` <br> **MORE DETAILS [BELOW TABLE](#notes-on-indirect-relative)**                                                                                                                                |
| 0xA      | Indirect autoIncrement | `add (L)+`                   | `add (VAR1)+`                      | Same like above but after absolute address has been loaded into register, address *itself* in memory cell is incremented. <br> ```AD = VAR1```<br>```VAR1 += 1``` <br> `VAR1` is a **pointer**. So **pointer** is modified. Yes, we are crazy here, we do pointer arithmetics                                                                                                                        |
| 0xB      | Indirect autoDecrement | `add -(L)`                   | `add -(VAR1)`                      | ```VAR1 -= 1``` <br> ```AD = VAR1``` <br> Again, pointer is modified, **NOT** value it points to                                                                                                                                                                                                                                                                                                     |
| 0xC      | Displacement SP        | `add &N` <br> `add (sp + N)` | `add &0x4a` <br> `add (sp + 0x4a)` | TODO                                                                                                                                                                                                                                                                                                                                                                                                 |
| 0xF      | Direct Load            | `add #N`                     | `add #0xff`                        | Load specified value into `DR`. Then a command may decide what to do with operand, e.g. load it into `AC` or do something else. Only one byte value can be set with direct load. The sign of value bit-extends i.e. `0xfe` becomes `0xfffe` and `0x7f` becomes `0x007f`                                                                                                                              |

*Note:* more information in [methodical](https://se.ifmo.ru/documents/10180/38002/Методические+указания+к+выполнению+лабораторных+работ+и+рубежного+контроля+БЭВМ+2019+bcomp-ng.pdf/d5a1be02-ad3f-4c43-8032-a2a04d6db12e) **page 22** and **page 32**

### Notes on *Indirect relative*

Let's consider example:
```asm
org 0x10 ; this command is optional because bcomp places program into this address by default
POINTER: word 0x15 ; absolute address of operand
LD (POINTER)

org 0x15
word 0x45a9 ; actual operand
```
Here, `LD (POINTER)` LD will look like this in memory: `0xA8FE`.

`A` means that the command is `LD`, `8` denotes that addressing mode is *indirect relative*
and `FE` is offset. Since offsets are represented in two's complement form, `FE` means (-2).
As you remember, `IP` is already incremented so at this time it points to `0x11 + 0x1 = 0x12`. 
Therefore **LD** will look for content at address `0x12 - 0x2 = 0x10`. Content of that cell will be interpreted as *absolute address* `0x15`
of where to search for *operand*. So the value at address `0x15` (which is `0x45A9` in the example) will be loaded into *Accumulator register*.


## Command execution stages

*Note:* [0, 2] means from 0 to 2 inclusive on both ends

| Stage             | Amount of memory accesses |
| ----------------- | ------------------------- |
| instruction fetch | 1                         |
| address fetch     | [0, 2]                    |
| operand fetch     | 1                         |
| execution         | [0, 1]                    |
| interrupt         | TODO                      |

*Note:* operand fetch is skipped for commands, which have `1` in their **14** bit:

<code>0<b><em>1</em></b>00 0000 0000 0000</code> here `14` bit is ***highlighted*** and set to `1`


P.S. Pull requests are welcome
