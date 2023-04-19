# Micro-commands

### SORA

#### Description
Sums or And (thats why SORA <- S OR A <- Sum OR And)

When 11 bit (right most bit has position 00) of microcode is set to:

```
ADD: 0000 0000 0001 0000 1110 0000 1001 0000 0001 0001 
                                        ^
AND: 0000 0000 0001 0000 1100 0000 1001 1000 0001 0001
                                        ^
```

 - `0` -- **SUM** sums operands on right and left ALU inputs 
 - `1` -- **AND** computes logical AND from right and left ALU inputs



