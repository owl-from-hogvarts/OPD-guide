# Guide for BCOMP

### Disclaimer
**THIS IT NOT COMPREHENSIVE DESCRIPTION! USE OFFICIAL [METHODICAL](https://se.ifmo.ru/documents/10180/38002/Методические+указания+к+выполнению+лабораторных+работ+и+рубежного+контроля+БЭВМ+2019+bcomp-ng.pdf/d5a1be02-ad3f-4c43-8032-a2a04d6db12e) FOR MORE INFO!**

## Modes

BCOMP can be launched in next modes:
 - `gui` -- default mode (graphical interface)
 - `cli` -- command line interface
 - `nightmare` -- literally nightmare, don't launch or you will get mental trauma
 - `dual` -- launches bcomp in both `gui` and `cli` modes simultaneously (**recommended for tracing**)

To launch bcomp in certain mode specify flag `-Dmode=`*`mode_name`* and replace `mode_name` with one of the above. Example:
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
|  `org ADDRESS`  | guides the compiler to place next value <br> (be it a raw value or a command) into cell <br> with address `ADDRESS`
| `word 0x0000,0xffa4,0xfa` | Places specified value into memory as is. <br> `?` is equal to `0x0000` in this context |

*Note:* you can use any case of letters you desire: `0xfA51`, `0xFF`, `0xac` are all valid.

Here is simple example:

```asm
org 0x04f
VAR1: word 0x45a9
add 0xf
sub VAR1
```

## Load `asm` file into bcomp
BCOMP has additional parameter to load file with code: `-Dcode=`*`file`* where `file` is any valid path to existing file: `foobar.asm`, `./keklol`, `/home/foo/kek` are all valid.

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

You can also write assembly right in BCOMP
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



<!-- ## Addressing

When you see something like this `2`**`E`**`F5` the second letter (`E` here) is responsible for addressing mode. Look at the table below

| Hex code |  Name  | Example | Description |
|----------|--------|---------|-------------|
|   0x0    |Absolute| add 0xf | Add number from memory cell with address `0xf` |
|   0x -->
 