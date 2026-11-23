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

4.
a) 
    #include "msp.h"

    void UART2_Init(void)
    {
        // Hold the module in reset while configuring it (slide 43).
        EUSCI_A2->CTLW0 |= 0x0001U;

        // Swap the default UART2 pin mapping to match the homework.
        PMAP->KEYID = 0x2D52U;      
        P3MAP->PMAP_REGISTER2 = 0x0EU; 
        P3MAP->PMAP_REGISTER3 = 0x0DU; 
        PMAP->KEYID = 0x0000U;      

        // Select the primary peripheral function (slide 42).
        P3->SEL0 |= 0x0CU;           // P3.2 TX and P3.3 RX.
        P3->SEL1 &= ~0x0CU;

        // Start with only UCSWRST set; clear other UART options.
        EUSCI_A2->CTLW0 = 0x0001U;
        EUSCI_A2->CTLW0 &= ~0x8000U; /
        EUSCI_A2->CTLW0 |= 0x2000U;  
        EUSCI_A2->CTLW0 &= ~0x1000U; 
        EUSCI_A2->CTLW0 &= ~0x0800U; 
        EUSCI_A2->CTLW0 &= ~0x0600U; 
        EUSCI_A2->CTLW0 &= ~0x0100U; 
        EUSCI_A2->CTLW0 &= ~0x00C0U; 
        EUSCI_A2->CTLW0 |= 0x0080U; 

        EUSCI_A2->BRW = 0x0068U;    
        EUSCI_A2->MCTLW = 0x0000U;  
        EUSCI_A2->IE = 0x0000U;     

        EUSCI_A2->CTLW0 &= ~0x0001U; 
    }

b) 
---- high ____ low
0xF1 = 11110001, D7 - D0 (MSB):

       IDLE |START| D7  | D6  | D5  | D4  | D3  | D2  | D1  | D0  |STOP |
         1  |  0  |  1  |  1  |  1  |  1  |  0  |  0  |  0  |  1  |  1  |
TX     -----|_____|-----|-----|-----|-----|_____|_____|_____|-----|-----|

0x55 = 01010101, D7 - D0 (MSB):

       IDLE |START| D7  | D6  | D5  | D4  | D3  | D2  | D1  | D0  |STOP |IDLE
Level    1  |  0  |  0  |  1  |  0  |  1  |  0  |  1  |  0  |  1  |  1  | 1
TX     -----|_____|_____|-----|_____|-----|_____|-----|_____|-----|-----|----

5. eUSCI_B function and SPI compared with UART

The eUSCI_B module supports both synchronous SPI and I2C communication. 

Aspect           SPI                         UART
Synchronization  
SPI uses shared clock edges. UART uses a start edge and agreed baud rate for each frame.

Clocking
The SPI's controller supplies SCLK, whereas UART has no separate clock wire.

Signaling
With SPI usually SCLK, MOSI, MISO, and a CS per peripheral. With UART, TX and RX, with a common 
electrical reference.

Data framing     
CS and clock pulses bound transfers; no mandatory start/stop/parity bits. With UART, Start bit, data, optional
parity, and stop bit(s).

6.
a)
SCLK: Controller-generated clock that determines when bits change
  and when they are sampled.
CS: Chip select; enables a particular peripheral for a transaction. It is usually active low, as in this assignment.
MISO: Data from peripheral to controller. (Master In, Slave Out)
MOSI: Data from controller to peripheral. (Master Out, Slave In)

b) 
If multiple devices drive a shared MISO line, their
outputs can disagree, producing bus contention, invalid received data,
and potentially excessive output current.

7.
a) UCBxCTLW0: This register configures the SPI operating format, clock source, and
controller/peripheral role, and holds the module in reset during setup.

b) SPI configuration fields

Register   Configuration
UCSYNC     Set for synchronous operation.
UCMODE     00: three-wire SPI; 01: four-wire, STE high; 10: four-wire, STE low; 11: I2C.
UCMST      Set for controller/master, clear for slave.
UCMSB      Set for MSB first; clear for LSB first.
UC7BIT     Clear for 8 bits; set for 7 bits.
UCCKPL     Clear for idle-low clock; set for idle-high.
UCCKPH     Set to sample first edge; clear for second.
UCSWRST    Set while configuring; clear to operate.

c) Clock-source selection

UCSSEL, bits 7:6, selects the controller's source clock:

    Field value     Bit Address     Source
        00           0x0000         Reserved
        01           0x0040         ACLK
        10           0x0080         SMCLK
        11           0x00C0         SMCLK

8.
a)
    #include "msp.h"

    void EUSCI_B0_SPI_Init(void)
    {
        EUSCI_B0->CTLW0 |= 0x0001U;  // Hold module in reset.

        // Drive CS inactive before enabling its GPIO output.
        P9->OUT |= 0x01U;
        P9->SEL0 &= ~0x01U;
        P9->SEL1 &= ~0x01U;
        P9->DIR |= 0x01U;            // P9.0: active-low GPIO CS.

        // UCCKPH=1, UCCKPL=0: SPI mode 0.
        // UCMSB=1, UC7BIT=0, UCMST=1, UCMODE=00, UCSYNC=1.
        // UCSSEL=10: SMCLK; UCSWRST=1 during configuration.
        EUSCI_B0->CTLW0 = 0xA981U;
        EUSCI_B0->BRW = 0x000CU;     // 12 MHz / 12 = 1 MHz.
        EUSCI_B0->IE = 0x0000U;      // Polling; interrupts disabled.

        P1->SEL1 &= ~0xE0U;
        P1->SEL0 |= 0xE0U;           // P1.5 CLK, P1.6 SIMO, P1.7 SOMI.

        EUSCI_B0->CTLW0 &= ~0x0001U; // Release reset; CTLW0 = 0xA980.
    }


b)
---- high ____ low
Bit              D7    D6    D5    D4    D3    D2    D1    D0
Value             0     1     0     1     0     1     1     1
CS       -----|__________________________________________________|-----
SCLK     ________|--|__|--|__|--|__|--|__|--|__|--|__|--|__|--|_________
MOSI     ___________|-----|_____|-----|_____|---------------------------
Sample           ^     ^     ^     ^     ^     ^     ^     ^
                 0     1     0     1     0     1     1     1
