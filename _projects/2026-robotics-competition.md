---
layout: project
title: Robotics Competition
description: Autonomous robot designed and built for a robotics competition focused on cube collection, sensor-based navigation, and mechanical expansion.
technologies: ["Arduino", "C++", "Sensors", "CAD", "Machining"]
image: /assets/images/robot-picture
---

Robotics Competition

![Photo of robotics competition robot]({{ "/assets/images/robot-picture" | relative_url }})

Robot Design and Strategy Overview

We designed our robot to fit within the required 8 in × 8 in starting area while expanding to a maximum width of 12 inches during operation to maximize cube collection. The robot combined a passive mechanical expansion system with sensor-based navigation.

Mechanically, the robot used a cardboard outer shell mounted onto a two-wheel DC motor chassis. The shell consisted of a back wall and two side flaps that acted as expandable collection walls. Cardboard was chosen because it was lightweight, durable enough for competition use, and easy to prototype and modify quickly.

At the start of the match, the side flaps were folded inward at 90° angles so the robot remained within the starting size constraint. Two 130° brackets attached to the outer sides of the flaps limited their motion once released, creating a V-shaped opening approximately 12 inches wide. To hold the flaps closed before activation, removable standoffs were placed between the brackets and back wall. Strings connected these standoffs to the wheels. As the robot began moving, the wheels rotated forward and pulled the strings, removing the standoffs and allowing the single-piece cardboard shell to spring outward naturally until constrained by the 130° brackets.

The expanded V-shaped opening allowed the robot to passively funnel and retain cubes while driving forward. Because the robot never drove backwards, collected cubes remained contained within the side walls.

Electrically, the robot used one centrally mounted color sensor and two front-mounted QTI sensors connected to an Arduino. The color sensor detected transitions between the blue and yellow sections of the field, helping identify when the robot crossed the center of the board. The two QTI sensors were positioned at the front of the robot to detect the black field borders and prevent the robot from driving off the platform.

During operation, the robot first drove forward to trigger the expansion mechanism. It continued moving until the color sensor detected a transition between the blue and yellow regions. The robot then turned 90° to the right and drove forward until the QTI sensors detected a black border. Upon reaching the border, the robot performed a 180° turn and continued driving toward the opposite side, repeating this back-and-forth motion for the remainder of the match.

---

Final Robot Design

**Figure 1.** Final robot design showing the starting configuration and expanded configuration after release.

![Robot design]({{ "/assets/images/robot-design" | relative_url }})

---

Design Process Reflection

The milestones helped us iterate through the robot’s basic design and functionality before developing the cube intake system. During early testing, we attempted to operate without QTI sensors, but the robot had difficulty detecting borders and would sometimes drift off the board. To solve this, we added two front-mounted QTI sensors, which significantly improved reliability.

Our mechanical design also evolved throughout the project. Initially, we designed an expanding arm mechanism to collect cubes in front of the robot, but it did not fit within the 12-inch diameter constraint. We then transitioned to an expanding flap design.

At first, we used polypropylene sheets, but they were too heavy, increased the risk of tipping, and did not spring open as effectively as slitted cardboard. They were also more difficult to prototype and modify after cutting. Because of this, we switched to cardboard, which was lighter, easier to iterate on, and more effective for the passive flap system.

During testing, we also found that collected cubes often jammed between the wheel and the robot wall, interfering with movement. To fix this, we added a wheel cage to prevent cubes from colliding with the wheels.

---

Competition Analysis

Our robot performed well during the competition and consistently collected between 8–13 cubes per match. The navigation sensors worked reliably, especially the QTI sensors, which allowed the robot to consistently stay on the board.

One unexpected issue involved turning calibration. During testing before the competition, the robot was under-turning, so we adjusted the timing to achieve a 90-degree turn. However, on competition day, the robot began over-turning, causing rotations greater than 90 degrees.

We also found that collisions with opposing robots sometimes caused cubes to fall out through the side flaps. In hindsight, it would have been beneficial to take fuller advantage of the allowed robot dimensions by extending the front of the robot with additional standoffs to create more separation between competing robots and the side flaps.

Additionally, our robot was slower than many competitors and experienced a delayed startup response after plugging in the battery.

Despite these issues, the robot’s strengths were its reliable border detection and dependable expansion mechanism. The expanding flaps consistently created a wide intake area that allowed efficient cube collection throughout the competition.

---

Conclusions

If we were to complete the project again under the same constraints and budget, we would focus more on optimizing speed and startup response time. We would also investigate the cause of the initial delay after battery connection and improve the robot’s reaction time. Larger wheels would likely improve overall movement speed as well.

For future students, we recommend prioritizing reliable border detection and consistent navigation before optimizing advanced features. Once the robot has dependable movement and intake functionality, improvements such as speed, turning calibration, and collection efficiency can significantly improve competition performance.

Small performance details became much more important during real matches against other robots than during isolated testing in the lab.

---

Appendix A: Bill of Materials (BOM)

| Item                                              | Quantity              | Unit Price   | Total Price |
|---------------------------------------------------|----------------------|--------------|--------------|
| Zinc-Plated Steel Surface Mount Hinges with Holes | 2                    | $10.75/pair  | $10.75       |
| Polypropylene Sheets                              | 1 (12” × 24” sheet)  | $6.98/sheet  | $6.98        |
| Corner Brackets                                   | 2                    | $1.06        | $2.12        |
| Cardboard Sheets                                  | 5 (36” × 36” sheets) | N/A          | N/A          |
| String                                            | 12 in                | N/A          | N/A          |

---

Appendix B: Circuit Diagram

![Circuit diagram]({{ "/assets/images/robot-schematic" | relative_url }})

---

Appendix C: Drawings

![Robot drawing]({{ "/assets/images/robot-drawing" | relative_url }})

---

Appendix D: Flowchart

![Robot flowchart]({{ "/assets/images/robot-flowchart" | relative_url }})

---

Appendix E: Code

```cpp
#include <avr/io.h>
#include <util/delay.h>
#include <avr/interrupt.h>

volatile int timerValue = 0;

// Interrupt for Color Sensor on PB4 (Pin 12)
ISR(PCINT0_vect) {
    if (PINB & 0b00010000) {
        TCNT1 = 0;
    } else {
        timerValue = TCNT1;
    }
}

uint16_t readQTI(uint8_t pinMask) {

    DDRD |= pinMask;
    PORTD |= pinMask;
    _delay_ms(1);

    DDRD &= ~pinMask;
    PORTD &= ~pinMask;

    uint16_t count = 0;

    while (PIND & pinMask) {
        count++;
        if (count > 30000) break;
    }

    return count;
}

int getColor() {
    PCMSK0 = 0b00010000;
    _delay_ms(10);
    PCMSK0 = 0b00000000;
    return (timerValue / 16);
}

void forward() { PORTB = 0b00001001; }
void back()    { PORTB = 0b00000110; }
void right()   { PORTB = 0b00000101; }
void left()    { PORTB = 0b00001010; }
void stop()    { PORTB = 0b00000000; }

int main(void) {

    UBRR0H = 0;
    UBRR0L = 103;
    UCSR0B = 0b00011000;
    UCSR0C = 0b00000110;

    DDRB = 0b00001111;
    DDRD = 0b00001100;

    TCCR1A = 0;
    TCCR1B = 0b00000001;

    PCICR |= 0b00000001;
    sei();

    _delay_ms(500);

    int startVal = getColor();

    while (1) {

        forward();

        int current = getColor();

        if(abs(current - startVal) > 5){
            break;
        }
    }

    stop();
    _delay_ms(500);

    right();
    _delay_ms(670);

    stop();
    _delay_ms(500);

    while (1) {

        uint16_t leftQTI = readQTI(0b01000000);
        uint16_t rightQTI = readQTI(0b10000000);

        if (leftQTI > 3000 || rightQTI > 3000) {

            stop();
            _delay_ms(200);

            right();
            _delay_ms(1675);

            stop();
            _delay_ms(200);

        } else {

            forward();
        }
    }
}
```
