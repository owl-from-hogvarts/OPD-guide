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
## Overflow

If value in memory cell is equal to `0x8000` or `0x8001` then loop will NOT jump! Due to how `loop` performs checks, Internal overflow occurs with these values. 