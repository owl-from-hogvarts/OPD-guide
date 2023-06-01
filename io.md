
# Input, Output

### The sixth bit mystery
SR (State register of External Device (ВУ - Внешнее устройство)) sets sixth (6) bit because setting 7 bit is affected by sign extension. Setting 0 - 3 bit may interfere with MR register (witch has the same address as SR).

### Transmission controller
*Transmits* data *to* external device

### Receiver controller
*Receives* data *from* external device
