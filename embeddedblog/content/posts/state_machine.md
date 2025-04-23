+++
title = 'State_machine'
date = 2025-04-21T17:24:06-03:00
draft = false
+++

My project today is just a little experiment on state processing. Just a green LED with a button that cycles it's levels from 0 to 3.

<!--more-->

![Circuit](/images/blink2.jpg)

The code is right here

``` c++
/****************************/
/*  LED level control       */
/*  Talkys Assis 2025-04-19 */
/****************************/

#define INPUT_PIN 4
#define OUTPUT_PIN 3

int power_level = 0;

void setup() {
  pinMode(INPUT_PIN, INPUT);
  pinMode(OUTPUT_PIN, OUTPUT);

}

void loop() {
    //just a loop to cycle from 0 to 3
    if(digitalRead(INPUT_PIN) == 1) {
        power_level++;
    }
    power_level = power_level % 4;

    analogWrite(OUTPUT_PIN, 50 * power_level); // Using 50 as step value
    delay(200);
}
```

Let's see it working:

![Circuit](/animations/blink2.webp)

Yeah, not that impressive, but I should note that using the delay as a debouncing tool was a smart move. But I really need a better debouncing setup.

— Talkys