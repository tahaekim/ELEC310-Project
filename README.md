# Generating PWM Waves Using an Astable Multivibrator

**ELEC 310 – Electronics Laboratory, Koç University**  
**Fall 2025 – Lab Project**

---

## Team

| Name | Role |
|------|------|
| Ahmet Taha Ekim | Circuit Design, Simulation & Implementation |
| Hasan Can Güneş | Circuit Design, Simulation & Implementation |

> Contact TA: Ekin Özgönül – eozgonul18@ku.edu.tr

---

## Overview

This is our term project for the ELEC 310 Electronics Lab course at Koç University. The goal was to design, simulate, and physically build a circuit that generates a **Pulse Width Modulated (PWM) signal** using a classic **astable multivibrator** topology, and then use that signal to **control the speed of a DC motor**.

The whole thing is built entirely out of discrete components — no ICs, no microcontrollers, no op-amps. Just transistors, resistors, capacitors, diodes, and LEDs on a breadboard.

We went through three design iterations:

1. A **basic astable multivibrator** to get the oscillation going and make sure we understood the fundamentals.
2. A **BJT-based PWM motor driver** that takes the multivibrator output and drives a power transistor (2N3055) to control a DC motor.
3. A **MOSFET-based PWM motor driver** that replaces the power BJT with an IRF510 N-channel MOSFET for better switching performance.

All three circuits were first simulated in LTSpice and then built on a breadboard for the demo.

---

## How It Works

### The Astable Multivibrator

The astable multivibrator is a free-running oscillator that continuously switches between two unstable states — it never settles. It's one of the most well-known transistor oscillator circuits out there.

The basic idea is this: two NPN transistors (2N3904 in our case) are cross-coupled through capacitors. When Q1 is ON (saturated), Q2 is OFF (cut-off), and vice versa. The capacitors charge and discharge through the base resistors, and once a capacitor charges enough to bring the opposite transistor's base above ~0.7V, the circuit flips to the other state. This keeps going back and forth indefinitely, producing a square wave at each collector.

The oscillation frequency is determined by:

$$f = \frac{1}{T} = \frac{1}{0.693 \times (R_2 \times C_1 + R_3 \times C_2)}$$

For a symmetric multivibrator (where R2 = R3 and C1 = C2), this simplifies to:

$$f = \frac{1}{1.386 \times R \times C}$$

The LEDs connected to each collector give a nice visual indication of the switching — they blink alternately.

### Making It a PWM Generator

To turn this into a PWM generator, we need to be able to adjust the **duty cycle** — i.e., how long the output stays HIGH vs. LOW in each period. In the basic version with equal R and C values on both sides, you get a ~50% duty cycle. That's not very useful for motor speed control.

The trick is to make the timing resistors **asymmetric**. By using different values for R2 and R3 (or by using potentiometers), we can independently control how long each half of the cycle takes. A larger R2 means C1 takes longer to charge, so Q2 stays OFF longer — increasing the duty cycle on one side.

In our PWM versions, we replaced the fixed base resistors with a potentiometer arrangement so you can smoothly vary the duty cycle from roughly 20% to 80% by just turning a knob. This directly controls the average voltage delivered to the motor and hence its speed.

### Motor Driver Stage

The astable multivibrator by itself can't deliver enough current to drive a motor. So we added an output stage:

- **BJT version:** The PWM signal from the multivibrator is fed (through a biasing network with R6, Q3, Q4) to a **2N3055 power NPN transistor** (Q5) configured as a switch. When the PWM signal is HIGH, Q5 saturates and current flows through the motor. When LOW, Q5 cuts off. A flyback diode (1N914) across the motor protects Q5 from inductive voltage spikes when it turns off.

- **MOSFET version:** Same concept, but the output stage uses an **IRF510 N-channel MOSFET** (M1) instead. The MOSFET has the advantage of a very high input impedance (voltage-driven gate), so it doesn't load the oscillator circuit as much. A gate pull-down resistor ensures the MOSFET turns off cleanly, and a flyback diode protects against back-EMF from the motor.

---

## Circuit Designs

### 1. Basic Astable Multivibrator (`Astable_Multivibrator.asc`)

The simplest version — just the oscillator with two LEDs.

| Component | Value | Purpose |
|-----------|-------|---------|
| Q1, Q2 | 2N3904 | NPN switching transistors |
| R1, R4 | 1 kΩ | Collector resistors (LED current limiting) |
| R2, R3 | 47 kΩ | Base/timing resistors |
| C1, C2 | 10 µF | Timing capacitors |
| D1, D2 | QTLP690C | LEDs for visual output |
| V1 | 9V | DC power supply |

**Estimated frequency:** f = 1 / (1.386 × 47k × 10µ) ≈ **1.53 Hz** (slow enough to see the LEDs blink)

### 2. PWM Motor Driver – BJT Version (`Astable_Multivibrator_pwm_bjt.asc`)

Astable multivibrator with adjustable duty cycle driving a 2N3055 power transistor for motor control.

| Component | Value | Purpose |
|-----------|-------|---------|
| Q1, Q2 | 2N3904 | Multivibrator transistors |
| Q3, Q4 | 2N3904 | Intermediate driver/buffer stage |
| Q5 | 2N3055 | Power transistor (motor driver) |
| R1, R4 | 1 kΩ | Collector resistors |
| R2, R3 | 50 kΩ | Timing resistors (potentiometers) |
| R5 | 4.7 kΩ | Bias resistor for center tap |
| R6 | 75 kΩ | Base resistor for Q3 |
| R7 | 100 Ω | Emitter resistor for Q4 |
| R8 | 2.2 kΩ | Collector resistor for R8 |
| R10 | 75 kΩ | Diode biasing resistor |
| C1, C2 | 100 nF | Timing capacitors |
| D1, D2 | QTLP690C | LEDs |
| D3, D4 | 1N914 | Flyback / protection diodes |
| DC_MOT | 50 Ω | DC motor (modeled as resistor) |
| V1 | 12V | DC power supply |

**Estimated frequency:** f = 1 / (1.386 × 50k × 100n) ≈ **144 Hz** (fast enough for smooth motor control)

### 3. PWM Motor Driver – MOSFET Version (`Astable_Multivibrator_pwm_mosfet.asc`)

Same oscillator concept but with an IRF510 MOSFET driving the load.

| Component | Value | Purpose |
|-----------|-------|---------|
| Q1, Q2 | 2N3904 | Multivibrator transistors |
| M1 | IRF510 | N-channel power MOSFET (motor driver) |
| R1, R4 | 1 kΩ | Collector resistors |
| R2 | 20 kΩ | Timing resistor (one side) |
| R3 | 80 kΩ | Timing resistor (other side) |
| R5 | 4.7 kΩ | Center tap bias resistor |
| R6 | 50 Ω | Motor load resistor |
| R7 | 100 kΩ | Gate drive resistor |
| R8 | 1 MΩ | Gate pull-down resistor |
| C1, C2 | 10 µF | Timing capacitors |
| D1, D2 | QTLP690C | LEDs |
| D3 | Diode | Flyback protection |
| V1 | 12V | DC power supply |

**Note:** R2 ≠ R3 here (20k vs 80k), so the duty cycle is intentionally asymmetric (~80%) to demonstrate speed control without a potentiometer.

---

## Repository Structure

```
ELEC310-Project/
├── README.md                                  ← You are here
├── LTSpice/
│   ├── Astable_Multivibrator.asc              ← Basic oscillator simulation
│   ├── Astable_Multivibrator_pwm_bjt.asc      ← PWM + BJT motor driver simulation
│   └── Astable_Multivibrator_pwm_mosfet.asc   ← PWM + MOSFET motor driver simulation
├── ELEC310-PROJECT.docx                       ← Project report
├── Elec310_Fall25_Project_Document (1).pdf    ← Course project manual
```

---

## How to Run the Simulations

1. **Install LTSpice** — it's free from [Analog Devices](https://www.analog.com/en/design-center/design-tools-and-calculators/ltspice-simulator.html). Available for Windows (and runs fine on macOS via Crossover/Wine or the native Mac version).
2. Open any of the `.asc` files from the `LTSpice/` folder.
3. Click **Run** (the running man icon) to start the transient simulation.
4. Probe the collector nodes of Q1/Q2 to see the oscillator waveforms, or the output node to see the PWM signal driving the motor.

For the basic multivibrator, the simulation runs for 3 seconds so you can observe several cycles of the blinking LEDs. The BJT PWM version runs for 50ms to show several PWM cycles at the higher frequency.

---

## Parts We Used

Here's the actual bill of materials we sourced for the breadboard build:

- 2× 2N3904 NPN transistors
- 1× 2N3055 NPN power transistor (for the BJT version) **or** 1× IRF510 N-channel MOSFET (for the MOSFET version)
- Assorted resistors (1kΩ, 4.7kΩ, 2.2kΩ, 47kΩ, 75kΩ, 100Ω)
- 2× 50kΩ potentiometers (for adjustable duty cycle)
- Capacitors (100nF ceramic, 10µF electrolytic)
- 2× LEDs
- 2× 1N914 signal diodes
- Small DC motor
- 9V / 12V power supply or battery
- Breadboard and jumper wires

Most of these parts can be found at electronics shops in Karaköy or from online retailers like [direnc.net](https://www.direnc.net), [Robotistan](https://www.robotistan.com), or [Özdisan](https://www.ozdisan.com).

---

## Design Notes & Lessons Learned

- **Start with the basic astable multivibrator.** Get that working first before adding the motor driver stage. It saves a lot of debugging time.
- **Potentiometers are your friend.** Even if you calculated exact resistor values, real-world component tolerances and transistor variations mean you'll need to tweak things. Having pots in the timing network lets you dial in the frequency and duty cycle on the fly.
- **The MOSFET version is cleaner for switching.** Since the IRF510 gate is voltage-driven, it doesn't draw current from the oscillator. The BJT version needed extra buffering stages (Q3, Q4) to avoid loading down the multivibrator.
- **Always include flyback diodes across inductive loads.** We learned this the hard way — without D3, the back-EMF from the motor when Q5 turns off can destroy the transistor.
- **Simulate before you build.** LTSpice saved us from a lot of wasted effort. Several component values that looked fine on paper didn't work in practice, and the simulation caught those issues early.
- **Capacitor values matter a lot.** Switching from 10µF to 100nF in the BJT version raised the frequency from ~1.5 Hz to ~144 Hz. Make sure you're in the right ballpark for your application.

---

## References

- Sedra & Smith, *Microelectronic Circuits* – Chapter on multivibrators and oscillator circuits.
- [LTSpice Documentation](https://www.analog.com/en/design-center/design-tools-and-calculators/ltspice-simulator.html)
- [Homemade Circuits – Transistor Projects](https://www.homemade-circuits.com/how-to-build-simple-transistor-circuits/)
- 2N3904 Datasheet – ON Semiconductor
- 2N3055 Datasheet – ON Semiconductor
- IRF510 Datasheet – Vishay Siliconix

---

## License

This project was done for educational purposes as part of the ELEC 310 course at Koç University. Feel free to use it as a reference, but please give credit if you base your work on ours.
