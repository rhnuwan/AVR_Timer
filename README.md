Timers and Interrupts
=====================

 

What is a timer?
----------------

  
A timer or to be more precise a timer / counter is a piece of hardware builtin
the Arduino controller  (other controllers have timer hardware, too). It is like
a clock, and can be used to measure time events.  
The timer can be programmed by some special registers. You can configure the
prescaler for the timer, or the mode of operation and many other things.   
The controller of the Arduino is the Atmel AVR ATmega168 or the ATmega328. These
chips are pin compatible and only differ in the size of internal memory. Both
have 3 timers, called timer0, timer1 and timer2. Timer0 and timer2 are 8bit
timer, where timer1 is a 16bit timer. The most important difference  between
8bit and 16bit timer is the timer resolution. 8bits means 256 values where 16bit
means 65536 values for higher resolution.  
The controller for the Arduino Mega series is the Atmel AVR ATmega1280 or the
ATmega2560.  Also identical only differs in memory size. These controllers have
6 timers. Timer 0, timer1 and timer2 are identical to the ATmega168/328. The
timer3, timer4 and timer5 are all 16bit timers, similar to timer1.  
All timers depends on the system clock of your Arduino system. Normally the
system clock is 16MHz, but for the Arduino Pro 3,3V it is 8Mhz. So be careful
when writing your own timer functions.   
The timer hardware can be configured with some special timer registers. In the
Arduino firmware all timers were configured to a 1kHz frequency and interrupts
are gerally enabled.  
  
Timer0:  
Timer0 is a 8bit timer.   
In the Arduino world timer0 is been used for the timer functions,
like [delay()](http://arduino.cc/en/Reference/Delay),[ millis()](http://arduino.cc/en/Reference/Millis) and[ micros()](http://arduino.cc/en/Reference/Micros).
If you change timer0 registers, this may influence the Arduino timer function.
So you should know what you are doing.   
  
Timer1:  
Timer1 is a 16bit timer.   
In the Arduino world the [Servo
library](http://arduino.cc/en/Reference/Servo) uses timer1 on Arduino Uno
(timer5 on Arduino Mega).  
  
Timer2:  
Timer2 is a 8bit timer like timer0.   
In the Arduino work the [tone()](http://arduino.cc/en/Reference/Tone) function
uses timer2.  
  
Timer3, Timer4, Timer5:  
Timer 3,4,5 are only available on Arduino Mega boards. These timers are all
16bit timers.  
  
  
Timer Register  
You can change the Timer behaviour through the timer register.  The most
important timer registers are:  
TCCRx - Timer/Counter Control Register. The prescaler can be configured here.

![](https://www.robotshop.com/letsmakerobots/files/userpics/u1433/tccr1a.jpg)

![](https://www.robotshop.com/letsmakerobots/files/userpics/u1433/tccr1b.jpg)

  
  
TCNTx - Timer/Counter Register. The actual timer value is stored here.  
  
OCRx - Output Compare Register  
  
ICRx - Input Capture Register (only for 16bit timer)  
  
TIMSKx - Timer/Counter Interrupt Mask Register. To enable/disable timer
interrupts.  
  
TIFRx - Timer/Counter Interrupt Flag Register. Indicates a pending timer
interrupt.

 

**Clock select and timer frequency**  


Different clock sources can be selected for each timer independently. To
calculate the timer frequency (for example 2Hz using timer1) you will need:

1.  CPU frequency 16Mhz for Arduino

2.  maximum timer counter value (256 for 8bit, 65536 for 16bit timer)

3.  Divide CPU frequency through the choosen prescaler (16000000 / 256 = 62500)

4.  Divide result through the desired frequency (62500 / 2Hz = 31250)

5.  Verify the result against the maximum timer counter value (31250 \< 65536
    success) if fail, choose bigger prescaler.

![](https://www.robotshop.com/letsmakerobots/files/userpics/u1433/prescaler.jpg)

**Timer modes**

Timers can be configured in different modes.

PWM mode. Pulth width modulation mode. the OCxy outputs are used to generate PWM
signals

CTC mode. Clear timer on compare match. When the timer counter reaches the
compare match register, the timer will be cleared

![](https://www.robotshop.com/letsmakerobots/files/userpics/u1433/waveform-gen.jpg)

What is an interrupt?

  
The program running on a controller is normally running sequentially instruction
by instruction. An interrupt is an external event that interrupts the running
program and runs a special interrupt service routine (ISR). After the ISR has
been finished, the running program is continued with the next instruction.
Instruction means a single machine instruction, not a line of C or C++ code.  
Before an pending interrupt will be able to call a ISR the following conditions
must be true:

-   Interrupts must be generally enabled

-   the according Interrupt mask must be enabled

  
Interrupts can generally enabled / disabled with the
function [interrupts()](http://arduino.cc/en/Reference/Interrupts) / [noInterrupts()](http://arduino.cc/en/Reference/NoInterrupts).
By default in the Arduino firmware interrupts are enabled. Interrupt masks are
enabled / disabled by setting / clearing bits in the Interrupt mask register
(TIMSKx).  
When an interrupt occurs, a flag in the interrupt flag register (TIFRx) is been
set. This interrupt will be automatically cleared when entering the ISR or by
manually clearing the bit in the interrupt flag register.  
  
The Arduino
functions[ attachInterrupt()](http://arduino.cc/en/Reference/AttachInterrupt) and[ detachInterrupt()](http://arduino.cc/en/Reference/DetachInterrupt) can
only be used for external interrupt pins. These are different interrupt sources,
not discussed here.   
  
Timer interrupts

  
A timer can generate different types of interrupts. The register and bit
definitions can be found in the processor data sheet
([Atmega328](http://www.atmel.com/dyn/resources/prod_documents/doc8271.pdf) or [Atmega2560](http://www.atmel.com/dyn/resources/prod_documents/doc2549.pdf))
and in the I/O definition header file (iomx8.h for Arduino, iomxx0_1.h for
Arduino Mega in the hardware/tools/avr/include/avr folder). The suffix x stands
for the timer number (0..5), the suffix y stands for the output number (A,B,C),
for example TIMSK1 (timer1 interrupt mask register) or OCR2A (timer2 output
compare register A).   
  
Timer Overflow:

  
Timer overflow means the timer has reached is limit value. When a timer overflow
interrupt occurs, the timer overflow bit TOVx will be set in the interrupt flag
register TIFRx. When the timer overflow interrupt enable bit TOIEx in the
interrupt mask register TIMSKx is set, the timer overflow interrupt service
routine ISR(TIMERx_OVF_vect)  will be called.  
  
Output Compare Match:

  
When a output compare match interrupt occurs, the OCFxy flag will be set in the
interrupt flag register TIFRx . When the output compare interrupt enable bit
OCIExy in the interrupt mask register TIMSKx is set, the output compare match
interrupt service ISR(TIMERx_COMPy_vect) routine will be called.  
  
Timer Input Capture:

  
When a timer input capture interrupt occurs, the input capture flag bit ICFx
will be set in the interrupt flag register TIFRx. When the input capture
interrupt enable bit  ICIEx in the interrupt mask register TIMSKx is set, the
timer input capture interrupt service routine ISR(TIMERx_CAPT_vect) will be
called.

 

PWM and timer

  
There is fixed relation between the timers and the PWM capable outputs. When you
look in the data sheet or the pinout of the processor these PWM capable pins
have names like OCRxA, OCRxB or OCRxC (where x means the timer number 0..5). The
PWM functionality is often shared with other pin functionality.   
The Arduino has 3Timers and 6 PWM output pins. The relation between timers and
PWM outputs is:  
Pins 5 and 6: controlled by timer0  
Pins 9 and 10: controlled by timer1  
Pins 11 and 3: controlled by timer2  
  
On the Arduino Mega we have 6 timers and 15 PWM outputs:  
Pins 4 and 13: controlled by timer0  
Pins 11 and 12: controlled by timer1  
Pins 9 and10: controlled by timer2  
Pin 2, 3 and 5: controlled by timer 3  
Pin 6, 7 and 8: controlled by timer 4  
Pin 46, 45 and 44:: controlled by timer 5

  
Usefull 3rd party libraries  
   
Some 3rd party libraries exists, that uses timers:

-   [Ken Shirrifs IR
    library](http://www.arcfn.com/2009/08/multi-protocol-infrared-remote-library.html).
    Using timer2. Send and receive any kind of IR remote signals like RC5, RC6,
    Sony  
    

-   [Timer1 and Timer3 library](http://arduino.cc/playground/Code/Timer1). Using
    timer1 or tiner3. The easy way to write your own timer interupt service
    routines.  
    

  
I have ported these libraries to different timers for Arduino Mega. All ported
libraries can be found in the attached file.

  
Pitfalls

  
There exists some pitfalls you may encounter, when programming your Arduino and
make use of functions or libraries that uses timers.

-   Servo Library uses Timer1. You can’t use PWM on Pin 9, 10 when you use the
    Servo Library on an Arduino. For Arduino Mega it is a bit more difficult.
    The timer needed depends on the number of servos. Each timer can handle 12
    servos. For the first 12 servos timer 5 will be used (loosing PWM on Pin
    44,45,46). For 24 Servos timer 5 and 1 will be used (loosing PWM on Pin
    11,12,44,45,46).. For 36 servos timer 5, 1 and 3 will be used (loosing PWM
    on Pin 2,3,5,11,12,44,45,46).. For 48 servos all 16bit timers 5,1,3 and 4
    will be used (loosing all PWM pins).

-   Pin 11 has shared functionality PWM and MOSI. MOSI is needed for the SPI
    interface, You can’t  use PWM on Pin 11 and the SPI interface at the same
    time on Arduino. On the Arduino Mega the SPI pins are on different pins.

-   tone() function uses at least timer2. You can’t use PWM on Pin 3,11 when you
    use the tone() function an Arduino and Pin 9,10 on Arduino Mega.
