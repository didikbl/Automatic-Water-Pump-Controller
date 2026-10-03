# Automatic Water Pump Controller Without a Microcontroller

An automatic water pump controller that detects the water level and switches the pump **ON or OFF automatically** using only logic ICs—without a microcontroller.

The system starts the pump when the water level falls below a preset threshold and stops it when the tank reaches **100%**. LEDs provide a visual indication of the water level.

## What We'll Build

This project combines:

* Water level detection
* Visual water-level indication
* Automatic pump control
* Manual pump control

The controller monitors four water levels:

| Probe | Water Level |
| ----- | ----------: |
| S1    |         25% |
| S2    |         50% |
| S3    |         75% |
| S4    |        100% |

When the water level falls below the selected threshold, the pump starts automatically. When the tank reaches 100%, the pump stops.

**No Arduino. No ESP32. No programming. Just simple electronic components and logic ICs.**

## What You'll Learn

By completing this project, you'll learn how to:

* Detect different water levels using simple probes and a **ULN2003**.
* Build an automatic water pump controller.
* Set a preset water level that triggers the pump.
* Create a visual water-level indicator.
* Build and test a complete control system without a microcontroller.

## Components

| Quantity | Component                           |
| -------: | ----------------------------------- |
|        1 | ULN2003 Darlington transistor array |
|        4 | Green LEDs                          |
|        4 | 1 kΩ resistors                      |
|        5 | 10 kΩ resistors                     |
|        1 | CD4071 quad 2-input OR gate         |
|        1 | CD4081 quad 2-input AND gate        |
|        1 | Electromagnetic relay               |
|        1 | Push button                         |
|        1 | Breadboard or Veroboard             |
|        — | Jumper/connecting wires             |

---

## ⚠️ Safety Warning

This project can be used to control a **220 V AC water pump**. Mains voltage is dangerous and potentially lethal.

* Never work on the 220 V side while the circuit is powered.
* Keep the low-voltage control circuit electrically isolated from the mains side.
* Use a relay rated for the pump's voltage and current.
* Properly insulate and enclose all mains connections.
* If you are not qualified to work with mains electricity, have the 220 V wiring handled by a qualified electrician.
* For initial testing, use a **low-voltage load** instead of a 220 V pump.

---

# How the Circuit Works

The system is divided into four main blocks:

```text
Water Level Detection
        ↓
Level Indication
        ↓
Pump Control Logic
        ↓
Pump Switching — Relay
```

## 1. Water Level Detection

The water level is detected using metal probes placed at different heights inside the tank.

Four wires connected to the inputs of the **ULN2003** act as level probes. A common electrode connected to **+12 V** is placed in the tank.

When water reaches a probe, it creates an electrical path between the +12 V common electrode and that probe. This activates the corresponding ULN2003 input.

| Probe | Water Level |
| ----- | ----------: |
| S1    |         25% |
| S2    |         50% |
| S3    |         75% |
| S4    |        100% |

## 2. Level Indication

The four corresponding ULN2003 outputs drive four green LEDs.

The LEDs are connected between **VCC** and the ULN2003 outputs. Because the ULN2003 outputs are active LOW, an LED turns ON when its corresponding output is pulled LOW.

| Water Level | ULN2003 Output | LED      |
| ----------- | -------------- | -------- |
| S1 — 25%    | LOW            | LED 1 ON |
| S2 — 50%    | LOW            | LED 2 ON |
| S3 — 75%    | LOW            | LED 3 ON |
| S4 — 100%   | LOW            | LED 4 ON |

As the water level falls, the upper probes lose contact with the water and their corresponding outputs return HIGH.

For example:

```text
Below 100% → Output 4 HIGH → LED 4 OFF
Below 75%  → Output 3 HIGH → LED 3 OFF
Below 50%  → Output 2 HIGH → LED 2 OFF
Below 25%  → Output 1 HIGH → LED 1 OFF
```

This provides a visual indication of the water level while also generating the logic signals used by the pump-control circuit.

---

# 3. Pump Control Logic

The pump control circuit uses one **OR gate** and one **AND gate**.

The control equation is:

```text
S_PUMP = S4 × (S2 + PUSH_BUTTON)
```

Where:

* `S2` = 50% water-level signal
* `S4` = 100% water-level signal
* `PUSH_BUTTON` = manual pump command
* `S_PUMP` = pump control signal

The OR gate first combines `S2` and `PUSH_BUTTON`.

The output of the OR gate is then combined with `S4` through the AND gate.

## Automatic Operation

When the water level falls below the preset **50% level**, `S2` becomes HIGH.

The OR gate therefore produces a HIGH output.

Because the tank is not full, `S4` is also HIGH. The AND gate consequently produces a HIGH `S_PUMP` signal, turning the pump ON.

As the tank fills and the water level rises above 50%, `S2` returns LOW. The feedback path keeps the pump command active while the tank is being filled.

When the water reaches the **100% probe**, `S4` becomes LOW. Since `S4` is an input of the AND gate, this forces `S_PUMP` LOW and turns the pump OFF regardless of the state of `S2` or the push button.

The automatic sequence is:

```text
Below 50%
    ↓
S2 HIGH
    ↓
Pump ON
    ↓
Water rises
    ↓
100% reached
    ↓
S4 LOW
    ↓
Pump OFF
```

## Manual Operation

The push button provides a manual way to start the pump.

When the button is pressed, the OR gate produces a HIGH output. The AND gate allows this command to reach `S_PUMP` only when `S4` is HIGH, meaning that the tank is not full.

Therefore:

```text
Tank not full + Push Button pressed
                ↓
             Pump ON
```

If the tank is already full, `S4` is LOW, so the AND gate blocks the command and the pump cannot be started.

### Changing the Automatic Starting Level

The automatic starting level can be changed by selecting a different level signal:

| Signal | Starting Level |
| ------ | -------------: |
| S1     |            25% |
| S2     |            50% |
| S3     |            75% |

The **100% probe S4 always remains the safety stop condition**.

---

# 4. Pump Control Truth Table

The pump-control logic is:

```text
S_PUMP = S4 × (S2 + PUSH_BUTTON)
```

| S4 | S2 | PUSH BUTTON | S2 + PUSH BUTTON | S_PUMP |
| -: | -: | ----------: | ---------------: | -----: |
|  0 |  0 |           0 |                0 |      0 |
|  0 |  0 |           1 |                1 |      0 |
|  0 |  1 |           0 |                1 |      0 |
|  0 |  1 |           1 |                1 |      0 |
|  1 |  0 |           0 |                0 |      0 |
|  1 |  0 |           1 |                1 |      1 |
|  1 |  1 |           0 |                1 |      1 |
|  1 |  1 |           1 |                1 |      1 |

Where:

* `S4 = 0` → tank is full → pump forced OFF.
* `S4 = 1` → tank is not full → pump can be activated.
* `S2 = 1` → water level is below 50%.
* `PUSH_BUTTON = 1` → manual start command.
* `S_PUMP = 1` → pump ON.
* `S_PUMP = 0` → pump OFF.

The `S4` signal therefore acts as an **enable signal** for the entire pump-control logic. Whenever the tank is full, the AND gate blocks the pump command.

---

# 5. Why Use the ULN2003?

The **ULN2003** provides the interface between the water-level probes and the logic circuit.

When water reaches a probe, it creates a conductive path between the +12 V common electrode and the probe. This produces a small current that activates the corresponding input of the ULN2003.

Each input controls a Darlington transistor inside the IC. When activated, the transistor pulls its output LOW.

```text
Water Probe
    ↓
ULN2003 Input
    ↓
Darlington Transistor
    ↓
LOW Output
```

The LOW output can then be used to:

* Turn ON the corresponding level LED.
* Provide a logic signal to the pump-control circuit.
* Handle multiple water-level probes using a single IC.

The ULN2003 contains **seven Darlington transistor channels**, allowing several level probes and indicators to be handled with relatively few components.

In this project, the ULN2003 performs two functions:

1. Water-level signal conditioning
2. Low-side switching for the level indicators

---

# 6. Pump Switching — Relay

The logic circuit cannot directly drive a 220 V AC pump.

An **electromagnetic relay** is therefore used as the interface between the low-voltage control circuit and the pump power circuit.

```text
Control Signal
      ↓
Relay Coil
      ↓
Relay Contacts
      ↓
Pump
```

The control signal energizes the relay coil, causing its contacts to switch the pump ON or OFF.

The relay provides electrical isolation between the low-voltage control electronics and the high-voltage pump circuit.

The relay must be properly rated for the pump's:

* Voltage
* Running current
* Starting current

All 220 V connections must also be properly insulated and enclosed.

---

# Step 3 — Install the Water-Level Probes

Install four water-level probes at different heights inside the tank.

| Probe   | Level |
| ------- | ----: |
| Probe 1 |   25% |
| Probe 2 |   50% |
| Probe 3 |   75% |
| Probe 4 |  100% |

For each level, install a common +12 V electrode slightly below its corresponding level probe.

When water reaches the probe, it creates a conductive path between the common electrode and the level probe.

The arrangement is:

```text
25%  → Common 1 + Probe 1
50%  → Common 2 + Probe 2
75%  → Common 3 + Probe 3
100% → Common 4 + Probe 4
```

Connect all common electrodes together and connect them to **+12 V**.

Each level probe has its own wire connected to the corresponding ULN2003 input.

This requires **five wires** between the tank and the control circuit:

* One +12 V common wire
* Four probe wires

Use corrosion-resistant conductive material for the electrodes. Keep the probes clean and properly spaced because water conductivity can vary depending on its mineral content.

---

# Step 4 — Build the Circuit

Build the electronic circuit according to the schematic.

Before applying power:

* Check all connections.
* Verify the polarity of the LEDs and other polarized components.
* Verify the logic connections.
* Verify the relay connections.
* Keep the 220 V section isolated from the low-voltage control circuit.

For the 220 V AC section, follow all safety precautions described above.

---

# Step 5 — Test the Circuit

Slowly fill the tank and verify that the LEDs turn ON as the water reaches each level:

```text
25%  → LED 1 ON
50%  → LED 2 ON
75%  → LED 3 ON
100% → LED 4 ON
```

Then slowly lower the water level and verify that the LEDs turn OFF in the reverse order.

Finally, test the pump-control system:

* The pump starts automatically when the water level falls below the preset level.
* The pump continues running while the tank is being refilled.
* The push button can manually start the pump when the tank is not full.
* The pump stops automatically when the tank reaches 100%.
* The pump cannot be started when the tank is already full.

If all these tests work as expected, the automatic water pump controller is ready for further testing and development.

---

# Step 6 — Limitations

Although this system is simple and practical, it has several limitations.

### Water Conductivity

The probes rely on the electrical conductivity of the water. Very low-conductivity water may result in unreliable level detection.

### Electrode Corrosion

Continuous contact with water can cause the electrodes to corrode over time. Corrosion-resistant materials are recommended.

### Probe Maintenance

The electrodes should be inspected and cleaned periodically to maintain reliable detection.

### Relay and Pump Rating

The relay must be properly rated for the pump, including its **starting current**.

### Mains Voltage

The 220 V AC section requires proper:

* Insulation
* Enclosure
* Protection
* Electrical installation

A qualified person should handle the mains installation when necessary.

---

# What's Next?

The controller can be upgraded with an **IR remote-control function**, allowing the pump to be controlled in three ways:

```text
             ┌─ Automatic — Water Level
             │
Pump Control ├─ Manual — Push Button
             │
             └─ Remote — IR Remote
```

An IR receiver and a simple logic stage can be added to the existing control circuit while keeping the automatic control system unchanged.

The **100% level condition must always remain the priority**: the pump must never be activated when the tank is full.
