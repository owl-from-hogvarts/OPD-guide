# LOOP

## Direct load
When used with direct load like:
```asm
loop #75
```

Will override itself with direct load content and transfer control to next command (as if value in memory cell was positive).
Before execution:
```
-> 8F75
   0700
   0100
```
After:
```
   0075
-> 0700
   0100
```

*Note:* other addressing modes should word as expected

## Indirect autoIncrement
This addressing mode ***will not break*** `loop` command. 
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

1. Decrement *address* stored in `Data Register` (`DR` for short) back. Notice, that `DR` is not cleared after address was incremented, so it remains in register

1. `loop` decrements operand `0x5` (located by address stored in `DR` i.e. `0x10`). (`0x5` -> `0x4`)

Infinite loop **DOES NOT** happen because `loop` performs on *operand* and **NOT** on the *address*. 

## Overflow

If value in memory cell is equal to `0x8000` or `0x8001` then loop will NOT jump! Due to how `loop` performs checks, Internal overflow occurs with these values. 