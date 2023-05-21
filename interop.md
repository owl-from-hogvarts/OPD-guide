
# Interrupt


Magic value `0x180` placed into second value of interrupt is loaded into `PS` register.
```asm
V1: word $default
    word 0x180  ; <- this magic value is loaded straight into PS
```

Let's look at it closer:
```
0x180
1 1000 0000
↑ ↑↑
| |└ disables interrupt
| └ WORK flag
└ PROGRAM flag
```
So this constant keeps BCOMP running (PROGRAM and WORK flags are required for that) and disables interrupt so our interrupt handler won't be interrupted. 

**Note**: later, when `IRET` would be executed, it will restore `PS` to state before interrupt has occurred. Thus the interrupt flag will be set back to `1` and interrupts will be enables again.
