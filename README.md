# ece528_fall26_homework3
# Section I: Review Questions


1. 
UCAxBRW: A 16-bit register containing the clock divider used to set the baud rate. 
UCAxMCTLW: Adjusts fractional baud-rate timing with UCBRF and UCBRS; UCOS16 enables 16x oversampling
UCAxCTLW: Configures parity, bit order, data length, stop bits, UART mode, and clock source. UCSWRST holds the module in reset during configuration
UCAxRXBUF: Holds received data for the program to read. Reading it also clears UCRXIFG register.
UCAxTXBUF: Holds data written by the program for transmission. The hardware shifts the data out serially.

2. 
The UCAxSTATW status register in the UART module reports activity and reception errors.
Bit 0 = Transmitting (UCBUSY)
Bit 6 = Framing error (UCFE)
Bit 5 = Overrun error (UCOE)
Bit 4 = Parity error (UCPE)
Bit 2 = 
    Flag       Bit    Mask       Meaning when set
    UCBUSY      0     0x0001     Transmitting OR receiving
    UCFE        6     0x0040     Framing error (invalid stop bit)
    UCOE        5     0x0020     Receive-buffer overrun
    UCPE        4     0x0010     Parity error
    UCRXERR     2     0x0004     At least one of these receive errors

Unread received data is indicated by UCRXIFG, bit 0 (0x0001) of UCAxIFG.

3. 

    N = f_SMCLK / baud = 12,000,000 / 115,200 = 104.1666667.

    UCA3BRW = 104 base 10 = 0x0068 or 0x0068U;

With oversampling and modulation disabled, the actual baud rate is:

    12,000,000 / 104 = 115,384.615 baud
