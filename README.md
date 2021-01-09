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

### Understanding the Microprocessor

I spent a lot of time with the datasheet for the 6502. My first goal was to get it to predictably execute NOP instructions. In other words, my first goal was to sucessfully make it to do nothing. 

In order to read the values on the data and address bus of the 6502, I would need a logic analyzer. I didn't have the means to obtain such a tool, so I decided to follow Ben Eater's approach here and use an Arduino MEGA to read the values of the address and data bus. This board had the number of digital inputs required to capture the value of all 8 data lines and all 16 addresses. The exact setup and code can be found on Ben Eater's website. I don't intend to reinvent the wheel here, so I won't be going over that.

![alt text](https://github.com/mxazfar/6502-Computer/blob/main/W65C02S%20Pinout.png?raw=true)

The first thing I did after setting up my makeshift logic analyzer was connect power and ground to the IC on pins 8 and 21 respectively on the pinout diagram above. Connecting up my logic analyzer, I saw nothing, as expected. There were many pins on the IC that needed to be set for normal operation.

First off, I needed to give the MPU a clock signal. I planned on using a one megahertz crystal oscillator for the clock; this would prove to be way too fast to be picked up by my logic analyzer, so I decided to cheaply hook up a button to the clock input to step through the program. The PHI2 pin on the MPU is the input for the clock cycle. The datasheet gives us the following information regarding the pin.

>Phase 2 In (PHI2) is the system clock input to the microprocessor internal clock. During the low power Standby Mode, PHI2 can be held in either high or low >state to preserve the contents of internal registers since the microprocessor is a fully static design.

This means that I could step through the MPU's program execution and observe the results slowly. After this, I took a look at the interrupt pins on the chip. The interrupt request line (IRQB) and the nonmaskable interrupt (NMIB) are both used to trigger interrupt events. These are active low signals; leaving the values floating could result in the processor continuously calling interrupts, so I decided to tie these pins high. 

Lastly, I tied the RDY and BE pins high as well. RDY allows the processor to run, and BE allows the data and address busses to output their values. 

After this, I powered the MPU back up and read the values off of the busses. Stepping through the clock, I saw what I expected. The microprocessor was hopping around and accessing various different addresses and reading and writing various different data. This is likely because there was no program; the MPU turned on, and executed the first instruction it read on the data lines. 

Looking through the datasheet, I found the machine code for the NOP instruction to be 0xEA. In binary, this translates to 0b11101010. I tied the data lines from D7 down to D0 through resistors to these values. After powering the MPU back on, I had accomplished my first goal. The processor was just walking through the address space of the computer. This is because the NOP instruction on the 6502 just tells it to check the next instruction. Tieing the data bus to 0xEA made the entire computer do nothing. The instruction took two clock cycles. This makes sense according to the datasheet. 

The NOP instruction only has one addressing method. The implied addressing method is used. 
