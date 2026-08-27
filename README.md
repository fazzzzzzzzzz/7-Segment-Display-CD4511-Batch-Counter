# Arduino 7 Segement Display & 3-Digit Batch Counter

An Arduino Uno batch/item counter (000–999) built for industrial production tracking. Three independent CD4511 BCD-to-7-segment decoders each drive their own digit, controlled by two debounced pushbuttons (with hold-to-repeat) and a potentiometer that acts as a variable-speed auto-count throttle.

## Features

- **3-digit display (000–999)** — hundreds, tens, and ones each driven by their own dedicated CD4511 decoder + common-cathode 7-segment display (no multiplexing, no flicker)
- **Increment / decrement buttons** — tap for ±1, hold past 400 ms to auto-repeat every 120 ms
- **Potentiometer throttle** — turn away from center to auto-count in that direction; the further from center, the faster it counts. A small dead zone around the middle keeps it stable when left alone
- **Clamped range** — count never overflows past 999 or underflows below 0
- Fully built and tested in Tinkercad

## Hardware

| Component | Qty |
|---|---|
| Arduino Uno R3 | 1 |
| CD4511 BCD-to-7-segment decoder | 3 |
| Common cathode 7-segment display | 3 |
| Pushbutton | 2 |
| Potentiometer (250 kΩ) | 1 |
| 200 Ω resistor | 3 |
| Breadboard | 1 |

## Pin Map

| Pin | Connected To |
|---|---|
| D2, D3, D4, D5 | Decoder 1 (hundreds) BCD inputs A, B, C, D |
| D6, D7, D8, D9 | Decoder 2 (tens) BCD inputs A, B, C, D |
| D10, D11, D12, D13 | Decoder 3 (ones) BCD inputs A, B, C, D |
| A0 | Decrement pushbutton (`INPUT_PULLUP`) |
| A1 | Increment pushbutton (`INPUT_PULLUP`) |
| A2 | Potentiometer wiper (analog input) |
| 5V | All 3 decoders' VDD, Lamp Test, Blanking; potentiometer terminal 1 |
| GND | All 3 decoders' GND, Latch Enable; potentiometer terminal 2; each display common (via 200 Ω resistor) |
| Decoder segment outputs (a–g) | Matching digit's own display segments (decoder 1 → display 1, decoder 2 → display 2, decoder 3 → display 3) |

## How It Works

Each loop cycle:
1. `handlePotentiometer()` checks how far the knob is turned from center. Inside a small dead zone, nothing happens. Outside it, the further the turn, the shorter the interval between automatic count changes.
2. `handleButton()` (called once per button) debounces the input, registers a single count change on press, and auto-repeats every 120 ms if held past 400 ms.
3. `changeCount()` applies the increment/decrement, clamped between 0 and 999.
4. `updateDisplays()` splits the count into hundreds/tens/ones and sends the correct 4-bit BCD value to each digit's decoder via `setBCD()`.

Full pseudocode, flowcharts, wiring diagrams, and a documented debug history are in `/docs`.

## Repository Structure

```
├── sketch.ino              Arduino source code
├── docs/
│   ├── Industrial_3Digit_Counter_Report.docx   Component list, spec, pseudocode, risk assessment, debug report, test plan
│   └── flowcharts/         Flowchart images
└── README.md
```

## Known Issues Fixed During Development

- Display power path exceeded safe current due to a resistor that never actually reached GND — rebuilt the common-to-resistor-to-GND path
- Displays stayed unlit due to a common anode vs. common cathode mismatch with the CD4511 — switched all displays to common cathode
- Displays stayed blank after the above fix due to Latch Enable being wired incorrectly — corrected to GND on all three decoders
- Potentiometer had no effect due to its outer terminals landing in signal rows instead of the power rails — rewired to the actual +5V/GND rails

Full details, including root cause and fix for each, are in the debug report inside `/docs`.

## License

For educational use.
