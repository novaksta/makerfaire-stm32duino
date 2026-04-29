# makerfaire-stm32duino

Repozitář obsahuje slajy a příklady z workshopu "STM32duino - posuňte své Arduino na vyšší úroveň" na makerfair v Praze 2026

## Blikaní LED, UART
```
void setup () {
	Serial.begin(115200);
	pinMode( PB7 , OUTPUT ) ;
	Serial.println("Hello!");
}

void loop () {
	digitalWrite( PB7, HIGH );
	delay (100) ;
	digitalWrite( PB7, LOW );
	delay (100) ;
}
```

## PWM
```
HardwareTimer htim4 = HardwareTimer ( TIM4 ); 
void setup () {
	pinMode( PB7 , OUTPUT ) ;
	 htim4.setPWM(2, PB_7, 5, 50) ; // Ch2, pin PB7 , 5Hz , 50% duty
}
void loop () {}
```

## PWM sinus -- proměnná, pole, cyklus 
### (optional)
```
HardwareTimer htim4 = HardwareTimer ( TIM4 ); 
uint8_t c = 0;
uint8_t sine[256]; 

void setup () {
pinMode(PB7 , OUTPUT ) ;
for ( int i =0; i <256; ++ i) 
sine[i]=( sin(i *2* PI /255.) +1.0)*100/2;
}
void loop () {
htim4.setPWM(2, PB7, 5000, sine[++c]) ; 
delay (5) ;
}
```
# I2C scan

```
#include <Wire.h>
TwoWire myI2C(PB9, PB8); //SDA, SCL
HardwareSerial mySerial(PG8, PG7);//Rx, Tx

void setup() {
	mySerial.begin(115200);
	myI2C.begin();
	myI2C.setClock(100000);
	for (uint8_t adresa = 1; adresa < 127; ++adresa) {
		myI2C.beginTransmission(adresa);
		if (!myI2C.endTransmission()) {//ACK LOW
			mySerial.print(“Found: 0x"); mySerial.println(adresa, HEX);}
	}
}
```

## USB mems myš
```
#include <ISM330DHCXSensor.h>
#define SerialPort  Serial
#define USE_I2C_INTERFACE
#define dev_interface       Wire
ISM330DHCXSensor AccGyr(&dev_interface);

#include "Mouse.h"
const PinMap PinMap_USB[] = {//patch PA_13 NOE
  {PA_11, USB, STM_PIN_DATA(STM_MODE_AF_PP, LL_GPIO_PULL_NO, GPIO_AF10_USB)}, // USB_DM
  {PA_12, USB, STM_PIN_DATA(STM_MODE_AF_PP, LL_GPIO_PULL_NO, GPIO_AF10_USB)}, // USB_DP
  {NC,    NP,  0} }; 
const int mouseButton = PC13;

void setup() {
  SerialPort.begin(115200);
  dev_interface.begin();
  AccGyr.begin();
  AccGyr.ACC_Enable();

	Mouse.begin();
  pinMode(mouseButton, INPUT);
}
 
void loop() {
  int32_t accelerometer[3];
  AccGyr.ACC_GetAxes(accelerometer);

  int  yDistance = accelerometer[0]/32;
	int  xDistance = accelerometer[1]/32;
	if ((xDistance != 0) || (yDistance != 0)) Mouse.move(xDistance, yDistance, 0); 
	if (digitalRead(mouseButton) == LOW) {
    if (Mouse.isPressed(MOUSE_LEFT)) Mouse.release(MOUSE_LEFT);
	} else {
    if (!Mouse.isPressed(MOUSE_LEFT)) Mouse.press(MOUSE_LEFT); 
	}
	delay(10);
}
```
