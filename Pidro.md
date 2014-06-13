# Pidro   


## What is Pidro

####Pidro is a remote control car (or we can consider it as a robot car since it was built to do something and can be control remotely) using Raspberry-Pi and a motor controller chips


## Some Basics that you need to know first 


* ![About GPIO](http://elinux.org/RPi_Low-level_peripherals#General_Purpose_Input.2FOutput_.28GPIO.29)
* ![GPIO set-up](http://learn.adafruit.com/adafruits-raspberry-pi-lesson-4-gpio-setup)
* ![TB6612FNG](http://www.pololu.com/product/713) dual motor controller


## Materials that you need 

* RaspberryPi
* ![TB6612FNG](http://www.pololu.com/product/713) dual motor controller
* 2pcs 6V DC motor (tb6612fng can handle up to 15v supply for dc motor please read the datasheet)
* 4pcs AA Battery
* battery holder for AA Battery
* Prototyping Board(Bread Board)
* Jumper Wire (Solid)
* Chassis and Wheel (you can buy some or you can custom on your own)


## Wiring Details (for GPIO to Motor-controller)

AIN1  =>  GPIO 27/21    PWMA  =>  17
AIN2  =>  GPIO 4        PWMB  =>  23
BIN1  =>  GPIO 24       STBY    =>  22
BIN2  =>  GPIO 25       VCC  =>  3.3v of RaspberryPi (pin 1)
AO1  =>  DC motor 1     VM  => power source (for DC motors)
AO2  =>  DC motor 1
BO1  =>  DC motor 2
BO2  =>  DC motor 2



* Pidgin for IM and Skype 

## Wiring Lay-out (the bastard diagram)

 





#@Chojayr 
