# Guide for BCOMP

### Disclaimer
**THIS IT NOT COMPREHENSIVE DESCRIPTION! USE OFFICIAL [METHODICAL](https://se.ifmo.ru/documents/10180/38002/Методические+указания+к+выполнению+лабораторных+работ+и+рубежного+контроля+БЭВМ+2019+bcomp-ng.pdf/d5a1be02-ad3f-4c43-8032-a2a04d6db12e) FOR MORE INFO!**

## Commands
- **[LOOP](loop.md)**

## Modes

BCOMP can be launched in next modes:
 - `gui` -- default mode (graphical interface)
 - `cli` -- command line interface
 - `decoder` -- dump default micro-program (microcode) into terminal
 - `nightmare` -- literally nightmare, don't launch or you will get mental trauma
 - `dual` -- launches bcomp in both `gui` and `cli` modes simultaneously (**recommended for tracing**)

To launch bcomp in certain mode specify flag <code>-Dmode=<em>mode_name</em></code> and replace `mode_name` with one of the above. Example:
```
java -jar -Dmode=dual bcomp-ng.jar
```

## Assembly
Assembly code can be written both in foreign text editors (like vscode and vim) or in `cli` mode.

In general, assembly is written alike below:
```
[LABEL: ] COMMAND_MNEMONIC ARGUMENT
```
*Note:* square brackets means that this part is optional. To see full syntax look at **page 51** in [methodical](https://se.ifmo.ru/documents/10180/38002/Методические+указания+к+выполнению+лабораторных+работ+и+рубежного+контроля+БЭВМ+2019+bcomp-ng.pdf/d5a1be02-ad3f-4c43-8032-a2a04d6db12e)

As mentioned above, assembly can be written in regular text files. Here some command that might come in handy:
|     Command     | Description |
|-----------------| ------------|
|  `org ADDRESS`  | guides the compiler to place next value <br> (be it a raw value or a command) into cell <br> with address `ADDRESS`. Commands and data following the command will be located one by one
| `word 0x0000,0xffa4,0xfa` | Places specified value into memory as is. <br> `?` is equal to `0x0000` in this context |

*Note:* you can use any case of letters you desire: `0xfA51`, `0xFF`, `0xac` are all valid.

Here is simple example:

```asm
org 0x04f
VAR1: word 0x45a9
add 0xf
sub VAR1
```
Here `0x45a9` will be located at `0x04f`, `add 0xf` at `0x050` and so on

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
Launch BCOMP in `dual` mode and load your program.
To see nice trace table in your terminal press `continue` button in GUI. You will see how neat lines appears in your terminal.

## Addressing

When you see something like this <code>2<strong>E</strong>F5</code> the second letter (`E` here) is responsible for addressing mode. Look at the table below (`L` stand for `Label`)

| Hex code | Name | Notation | Example | Description |
|----------|-------|----------|---------|---|
|  0x0-0x7 |Absolute| `add $L` <br> `add ADDR` | `add $VAR1` <br> `add 0xf` | Add number from memory cell with address `0xf` or from `$VAR1` label |
|    0xE   | Direct relative | `add L` | `add VAR1` <br> `4EFE` | **Only labels are supported!** `IP + 1 + OFFSET`. Notice, that `IP` point to *next* command. So that is where `+ 1` comes from. Offset can be *positive* and *negative*. Something  like `0x80`, `0xfe` are **negative**. Let's assume that `4EFE` (`add`) have address `0x010`. Therefore it points to address right before `4EFE` (`0xFE` is *negative* = `-2`) i.e. `0x009`. `4EFF` point to itself
|    0x8   | Indirect relative | `add (L)` | `add (VAR1)` | Works like pointers in C/C++. It's like saying: *Hey, look in this box. Here you find paper which tells you where exactly the thing is.* `add` --(Direct relative)--> `VAR1` --(Absolute)--> `value` <br> **MORE DETAILS BELOW TABLE**|
|    0xA   | Indirect autoIncrement | `add (L)+` | `add (VAR1)+` | Same like above but after absolute address has been loaded into register, address *itself* in memory cell is incremented. <br> ```AD = VAR1```<br>```VAR1 += 1``` <br> `VAR1` is a **pointer**. So **pointer** is modified. Yes, we are crazy here, we do pointer arithmetics |
|    0xB   | Indirect autoDecrement | `add -(L)` | `add -(VAR1)` | ```VAR1 -= 1``` <br> ```AD = VAR1``` <br> Again, pointer is modified, **NOT** value it points to |
|    0xC   | Displacement SP | `add &N` <br> `add (sp + N)` | `add &0x4a` <br> `add (sp + 0x4a)` | TODO |
|    0xF   | Direct Load | `add #N` | `add #0xff` | Load specified value into `AC`. Only one byte value can be set with direct load. The sign of value bit-extends i.e. `0xfe` becomes `0xfffe` and `0x7f` becomes `0x007f`

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
Here, `LD (POINTER)` will result in next data in memory: `0xA8FE`.

`A` means that the command is `LD`, `8` denotes that addressing mode is *indirect relative*
and `FE` is offset. Since offset are represented in two's complement form, `FE` means (-2).
As you remember, `IP` is already incremented so at the time it points to `0x11 + 0x1 = 0x12`. 
Therefore **LD** will interpret content at `0x12 - 0x2 = 0x10` address as *absolute address* (`0x15` is *absolute address*)
of where *operand* (`0x45A9` in this example, the thing which actually will be loaded into *Accumulator register*) is stored.


P.S. Pull requests are welcome
