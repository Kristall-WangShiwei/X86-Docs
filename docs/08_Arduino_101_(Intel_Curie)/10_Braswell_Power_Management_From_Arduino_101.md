The Arduino 101 side (Intel® Curie™) of the UDOO X86 can take care of the Braswell power management.  
It can act as the Power Button(PWR) of the board connected directly to the Braswell processor. This means that an Arduino 101 sketch, that it's always running while the board is powered up, can for example turn on/off the Braswell main processor or wake the OS from the suspend state.

The Arduino 101 on board can wake up the Braswell processor depending on various triggers. For example, it can wake up the Braswell prcessor if the temperature, registered by a connected temperature sensor, increases over a certain threshold. Other examples of triggers may be a BLE signal or a movement tracked by the 6-axis motion sensor on board, defining features of Arduino 101. By exploiting the Arduino capabilities, you can set up the trigger you want to get the behaviour you expect from the system.

The Arduino 101 (Intel® Curie™) can trigger a power signal of the Braswell processor by producing a **20ms low pulse** in the **Arduino Pin 9** (IO9/PWM3 signal of the schematics).

You need to enable this feature of the UDOO X86 board in the BIOS setup(SCU). You can find the **Curie Power Management** option in the menu **Power**:

    Power

    ...
    Curie Power Management    <Enabled>
    Power On Intel Curie      <Enabled>


The option you can choose for the **Curie Power Management** are:

    <Disabled>    =   Intel Curie will never change the power status of the system

    <Wake Only>   =   Intel Curie only be able to wake the system from S3/S4/S5

    <Enabled>     =   Intel Curie will be able to put the system in a lower power
                      state(S3/S4/S5 depending on OS configuration) and wake it

## Example

Following an example of an Arduino sketch that could be used to trigger a power signal from the Arduino 101 (Intel® Curie™) when a button is pressed:

Circuit design:
<TODO circuit image>


Arduino sketch:
```
int reset_pin = 9;    /* Triggers the power signal */
int input_pin = 2;    /* Input Button connected. Set in pull down with a resistor */

int pulse_time = 20;

void setup() {
  pinMode(reset_pin, OUTPUT);
  digitalWrite(reset_pin, HIGH);

  pinMode(input_pin, INPUT);
}

void loop() {
  /* When the button (connected to the Pin 2) is pressed, the Pin 9  
  *  goes LOW for 20ms.
  */
  if (digitalRead(input_pin) == HIGH) {
    digitalWrite(reset_pin, LOW);
    delay(pulse_time);
    digitalWrite(reset_pin, HIGH);
  }
  delay(10);
}
```