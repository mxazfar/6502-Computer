# 6502-Computer
Documentation of the design and development of a 6502 MPU-based computer. 

This computer is very heavily inspired by Ben Eater's series of videos on the [6502 microprocessor](https://www.youtube.com/user/eaterbc).

# Inspiration
After completing the first semester of my sophomore year as an electrical engineering student, I wanted to find a way to apply what I had learned so far. This project combined everything I had learned so far in my formal education, along with satsifying the curiosities I had regarding how exactly what I learned would come into play in the engineering design process.

So far, the biggest takeaway from this for me has been the actual design process.

# Designing the Computer

The Western Design Center is the sole producer of the 6502 microprocessor. They also produce other interfacing ICs to help facilitate the I/O functions of the processor using the interrupts located on the processor.

### The Parts
- W65C02S6TPG-14 [Datasheet] and [Jameco Link]  
  This served as the Central Processing Unit (CPU) of the computer. This IC had full control of the computer other than interrupt handling.

- AT28C256 [Datasheet] and [Jameco Link]   
  This 32KB EEPROM was used to store the contents of the program that would run on the computer. 
  
- HM62256LP-70 [Datasheet] and [Jameco Link]  
  The Randomly Accessed Memory (RAM) of this computer was stored on this chip. It is essentially the same as the EEPROM, only it is volatile and erases once power is cut off.
  
- W65C22S6TPG-14 [Datasheet] and [Jameco Link]  
  All I/O was handled by this Versatile Interface Adapter (VIA). This included interfacing with the LCD display and handling interrupts.

- 16x2 Character LCD [Datasheet] and [Jameco Link]  
  This display was used to produce human-readable output. It was interfaced to using the VIA. The datasheet for the actual interface adapter on the LCD module can be found [here]. 
