# LOOP

Command LOOP has actual name: "декремент и пропуск"

Here you will find at least 3 ways how to break the `loop` command

## Direct load
When used with direct load like:
```asm
loop #75
```

Loop will override itself with direct load content decremented and transfer control dependent on payload. 
Thus if positive number (e.g. `0x75`) is provided as direct load, *loop* will transfer control to next command as if value in memory cell was positive.
And if payload is negative (e.g. `0xF5`) loop will behave like with negative value in memory cell i.e. jump over next command.

(Positive payload) Before execution:
```
-> 8F75
   0700
   0100
```
After:
```
   0074
-> 0700
   0100
```

(Negative payload) Before execution:
```
-> 8FF5
   0700
   0100
```
After execution:
```
   FFF5 ; sign is bit extended, therefore you see FFF5 instead of 00F5
   0700
-> 0100
```

*Note:* other addressing modes should work as expected

## Indirect autoIncrement

### Normal
This addressing mode will *not* break `loop` command if used as below:
```asm
org 0x10
word 0x5           ; <- operand
POINTER: word 0x10 ; <- address
loop (POINTER)+    ; <- autoIncrement
inc
hlt
```
1. Increment *address* and place it into `Data Register`

1. Store `Data Register` content (i.e. *incremented address*) to `POINTER` (`0x10 -> 0x11`)

1. Decrement *address* stored in `Data Register` (`DR` for short) back (`0x11 -> 0x10`). Notice, that `DR` has not been cleared cleared after address increment, so address remains in register

1. `loop` decrements operand `0x5` (located at address stored in `DR` i.e. `0x10`). (`0x5` -> `0x4`)

Infinite loop **DOES NOT** happen because `loop` performs decrement on *operand* and **NOT** on the *address*. 

### The WTF
But wait! What... What if *address* and *operand* were the same essence.
So... Here it is:
```asm
org 0x10
POINTER: word 0x10 ; notice pointer points to itself
LP: loop (POINTER)+
jump LP
hlt
```
Please! Don't run it as this will break universe to irrecoverable state!

So that's how we trigger infinite *loop*. 

`0x10 -> 0x11 -> 0x10` and so on.

 *Notice* that increment happens first, because *address increment* belongs to *address fetch* phase. In turn, *operand decrement* happens during *Execution* phase. *Address fetch* precedes *Execution*. 

## Overflow

If value in memory cell is equal to `0x8000` or `0x8001` then loop will NOT jump! Due to how `loop` performs checks, Internal overflow occurs with these values. 