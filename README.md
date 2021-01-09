# 6502-Computer
Documentation of the design and development of a 6502 MPU-based computer. 

This computer is very heavily inspired by Ben Eater's series of videos on the [6502 microprocessor](https://www.youtube.com/user/eaterbc).

# Inspiration
After completing the first semester of my sophomore year as an electrical engineering student, I wanted to find a way to apply what I had learned so far. This project combined everything I had learned so far in my formal education, along with satsifying the curiosities I had regarding how exactly what I learned would come into play in the engineering design process.

So far, the biggest takeaway from this for me has been the actual design process.

# Designing the Computer

The Western Design Center is the sole producer of the 6502 microprocessor. They also produce other interfacing ICs to help facilitate the I/O functions of the processor using the interrupts located on the processor.

## The Parts
- W65C02S6TPG-14 [Datasheet](https://www.westerndesigncenter.com/wdc/documentation/w65c02s.pdf) and [Jameco Link](https://www.jameco.com/z/W65C02S6TPG-14-Western-Design-Center-MPU-8-Bit-14MHz-65KB-Memory-40-Pin-PDIP_2143638.html)  
  This served as the Central Processing Unit (CPU) of the computer. This IC had full control of the computer other than interrupt handling.

- AT28C256 [Datasheet](https://ww1.microchip.com/downloads/en/DeviceDoc/AT28C256-–-Industrial-Grade-256-Kbit-(32,768-x-8)-Paged-Parallel-EEPROM.pdf) and [Jameco Link](https://www.jameco.com/z/28C256-15-Major-Brands-IC-28C256-15-EEPROM-256K-Bit-CMOS-Parallel_74843.html)   
  This 32KB EEPROM was used to store the contents of the program that would run on the computer. 
  
- HM62256LP-70 [Datasheet](https://www.jameco.com/Jameco/Products/ProdDS/82472.pdf) and [Jameco Link](https://www.jameco.com/z/62256LP-70-Major-Brands-IC-62256LP-CMOS-SRAM-256K-Bit-32Kx8-70ns-Low-Power_82472.html)  
  The Randomly Accessed Memory (RAM) of this computer was stored on this chip. It is essentially the same as the EEPROM, only it is volatile and erases once power is cut off.
  
- W65C22S6TPG-14 [Datasheet](https://www.westerndesigncenter.com/wdc/documentation/w65c22.pdf) and [Jameco Link](https://www.jameco.com/z/W65C22S6TPG-14-Western-Design-Center-Versatile-Interface-Adapter-via-8-Bit-I-O-Ports-14-MHz-40-Pin-PDIP-CMOS-5-Volt_2143591.html)  
  All I/O was handled by this Versatile Interface Adapter (VIA). This included interfacing with the LCD display and handling interrupts.

- 16x2 Character LCD [Datasheet](https://www.jameco.com/Jameco/Products/ProdDS/2160374.pdf) and [Jameco Link](https://www.jameco.com/z/FIT0127-DFRobot-16x2-Character-LCD-Display-White-on-Blue-5V_2160374.html)  
  This display was used to produce human-readable output. It was interfaced to using the VIA. The datasheet for the actual interface adapter on the LCD module can be found [here](https://www.sparkfun.com/datasheets/LCD/HD44780.pdf). 
  
These are not all of the parts that are used. The various modules that are discussed below require their own parts and configurations; the list and datasheets for the parts are included with the documentation of the modules.
  
## The Process

As previously stated, this project follows Ben Eater's general 6502 layout closely. Given the nature of 8-bit computing, there really aren't many wildly different hardware configurations. The program essentially has to be stored on an EEPROM given the addressing of the 6502. The selection of the RAM chip is more or less arbitrary, but this specific IC worked well because it's pinout is the exact same as the 28C256. This made the breadboard prototyping stage a lot easier, which was the bulk of the process. The VIA is pretty much essential, as it is designed to work with the microprocessor and presents no timing issues or other complications. The selection of the LCD has some leeway, as it is purely arbitrary and solely used for human-readable output; if that isn't required, then the LCD module is pointless and can be removed. 

Even though I used the same hardware, my process was different from Ben Eater's. I took a hierarchical and modular approach to the development of the computer, setting goals for myself and building those goals as I progressed through the build. 

### Understanding The Microprocessor

I spent a lot of time with the datasheet for the 6502. My first goal was to get it to predictably execute NOP instructions. In other words, my first goal was to sucessfully make it to do nothing. 

In order to read the values on the data and address bus of the 6502, I would need a logic analyzer. I didn't have the means to obtain such a tool, so I decided to follow Ben Eater's approach here and use an Arduino MEGA to read the values of the address and data bus. This board had the number of digital inputs required to capture the value of all 8 data lines and all 16 addresses. The exact setup and code can be found on Ben Eater's website. I don't intend to reinvent the wheel here, so I won't be going over that.

![alt text](https://github.com/mxazfar/6502-Computer/blob/main/W65C02S%20Pinout.png?raw=true)

The first thing I did after setting up my makeshift logic analyzer was connect power and ground to the IC on pins 8 and 21 respectively on the pinout diagram above. Connecting up my logic analyzer, I saw nothing, as expected. There were many pins on the IC that needed to be set for normal operation.

First off, I needed to give the MPU a clock signal. I planned on using a one megahertz crystal oscillator for the clock; this would prove to be way too fast to be picked up by my logic analyzer, so I decided to cheaply hook up a button to the clock input to step through the program. The PHI2 pin on the MPU is the input for the clock cycle. The datasheet gives us the following information regarding the pin.

>Phase 2 In (PHI2) is the system clock input to the microprocessor internal clock. During the low power Standby Mode, PHI2 can be held in either high or low state to preserve the contents of internal registers since the microprocessor is a fully static design.

This means that I could step through the MPU's program execution and observe the results slowly. After this, I took a look at the interrupt pins on the chip. The interrupt request line (IRQB) and the nonmaskable interrupt (NMIB) are both used to trigger interrupt events. These are active low signals; leaving the values floating could result in the processor continuously calling interrupts, so I decided to tie these pins high. 

Lastly, I tied the RDY and BE pins high as well. RDY allows the processor to run, and BE allows the data and address busses to output their values. 

After this, I powered the MPU back up and read the values off of the busses. Stepping through the clock, I saw what I expected. The microprocessor was hopping around and accessing various different addresses and reading and writing various different data. This is likely because there was no program; the MPU turned on, and executed the first instruction it read on the data lines. 

Looking through the datasheet, I found the machine code for the NOP instruction to be 0xEA. In binary, this translates to 0b11101010. I tied the data lines from D7 down to D0 through resistors to these values. After powering the MPU back on, I had accomplished my first goal. The processor was just walking through the address space of the computer. This is because the NOP instruction on the 6502 just tells it to check the next instruction. Tieing the data bus to 0xEA made the entire computer do nothing. The instruction took two clock cycles. This makes sense according to the datasheet. 

The NOP instruction only has one addressing method. The implied addressing method is used. The datasheet helps shed light on this addressing method.

![alt text](https://github.com/mxazfar/6502-Computer/blob/main/Implied%20addressing.png?raw=true)

This tells us that the implied addressing method takes in just the opcode and no other inputs. The addressing mode table can give us some insight as to why the instruction takes two clock cycles.

![alt text](https://github.com/mxazfar/6502-Computer/blob/main/Addressing%20Mode%20Table.png?raw=true)

Looking at the instruction time in memory cycles, we can see that the implied addressing takes two clock cycles. This can dispel all concerns regarding errors. 

This approach of hard coding the data values using resistors worked to certify that my hardware setup up to this point was correct. My next goal was to get some human-readable output from the machine. 

### Introducing A ROM

I needed a place to store more complex code. This is where the ROM came into play.

Before I could use assembly language, I decided to use python to create a binary image containing the code I needed. The code for the script can be seen below.

```python
rom = bytearray([0xea] * 32768)

with open("rom.bin", "w") as out_file:
    out_file.write(rom);
```

The pinout of the EEPROM can be seen below.

![alt text](https://github.com/mxazfar/6502-Computer/blob/main/AT28C256%20Pinout.png?raw=true)

After connecting power and ground, I connected the data bus and address bus of the MPU to the busses on the EEPROM. I also decided to tie the Write Enable pin high, as I wasn't going to write to the ROM while it was in the computer at any point. 

Connecting power to the computer, I was surprised to see processor acting unpredictably yet again. This wasn't entirely unsuspected. The chip enable pin on the EEPROM was not tied to anything, so it was not outputting anything. After tying the pin low, the processor worked exactly as expected for all addresses that were less than 0x8000. This made sense because the EEPROM is 32KB, which would contain addresses 0x0000 through 0x7FFF.

While this worked to get the MPU to do nothing, this was not a long term solution. I had not considered the reset pin of the MPU at all; it had been tied high in the initial hardware setup, so the processor just started from a random spot whenever it was turned on. The datasheet can give us some information on the reset pin. 

>The Reset (RESB) input is used to initialize the microprocessor and start program execution. The RESB signal must be held low for at least two clock cycles after VDD reaches operating voltage. Ready (RDY) has no effect while RESB is being held low. All Registers are initialized by software except the Decimal and Interrupt disable mode select bits of the Processor Status Register (P) are initialized by hardware. When a positive edge is detected, there will be a reset sequence lasting seven clock cycles. The program counter is loaded with the reset vector from locations FFFC (low byte) and FFFD (high byte). This is the start location for program control. RESB should be held high after reset for normal operation.

The first thing I took from this was that I needed to include the addresses 0xFFFC and 0xFFFD in my EEPROM's range so that it could tell the processor where the reset vector is pointing. This was as simple as inverting and connecting the 16th address pin of the MPU to the write enable pin on the EEPROM. The 74LS04 IC was used to invert the signal (datasheet can be found [here](https://www.ti.com/lit/ds/symlink/sn74ls04.pdf)). This resulted in the EEPROM being active only when the 6502 was reading from addresses 0x8000 and 0xFFFF. In return, this satisfied our requirement of including the reset vector in the ROM. I also decided to hook up a button to the RESB pin on the processor so I could toggle resets. 

My code for filling the EEPROM had to be edited to add the reset vector. I needed to specify the reset vector as 0x8000 to ensure that the processor doesn't leave the range of the EEPROM. The processor is little endian, meaning it takes in the lower byte of the address first and the higher byte second. I had to store these at what the processor would view as 0xFFFC and 0xFFFD. Since the entire address space was shifted up, I had to write these values to 0x7FFC and 0x7FFD in the EEPROM. The python code below does a better job of explaining. 

```python
rom = bytearray([0xea] * 32768)

rom[0x7ffc] = 0x00
rom[0x7ffd] = 0x80

with open("rom.bin", "w") as out_file:
    out_file.write(rom);
```

After writing this to the EEPROM and powering up the computer, the output behaved as expected after the reset button was pushed. This told me that the processor is able to retrieve the location of the reset vector from the ROM correctly, ensuring that the ROM had been set up correctly.

### Blinking Some LEDs
