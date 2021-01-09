# 6502-Computer
Documentation of the design and development of a 6502 MPU-based computer. 

This computer is very heavily inspired by Ben Eater's series of videos on the [6502 microprocessor](https://www.youtube.com/user/eaterbc).

# Inspiration
After completing the first semester of my sophomore year as an electrical engineering student, I wanted to find a way to apply what I had learned so far. This project combined everything I had learned so far in my formal education, along with satsifying the curiosities I had regarding how exactly what I learned would come into play in the engineering design process.

So far, the biggest takeaway from this for me has been the actual design process. This is currently being documented in its entirety in the wiki section of this repo. 

# Todo

## Repository 

- [ ] Complete documentation of design process
- [ ] Complete documentation of modules

## Project

### Digital Clock Module

- [ ] Calculate appropriate resistance values for 555 timer configuration
- [ ] Implement days/weeks/years in assembly program
- [ ] Calculate clock drift analytically and observationally

### Interrupt Handling

- [ ] Acquire 74LS148 (8 to 3 encoder) to expand hardware interrupts
- [ ] Implement interrupt expansion in assembly using the VIA's interrupt flags register
