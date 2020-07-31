# Rotary Encoders
## Pinout
```
-- Clock
-- Data
-- Switch
-- Vcc
-- Gnd
```
## Traces
The following are traces taken rotating the encoder in both directions.

###### Rotating clockwise

![Clockwise](docs/CW-Yellow-Clock--Green-Data.PNG)

In clockwise rotation, the clock pulse happens first, then the data pulse. In the above trace, the yellow probe is connected to the clock pin, and the green probe is connected to the data pin.

###### Rotating counter-clockwise

![Counter-clockwise](docs/CCW-Yellow-Clock--Green-Data.PNG)

In counter-clockwise rotation, the data pulse happens first, then the clock pulse. In the above trace, the yellow probe is connected to the clock pin, and the green probe is connected to the data pin.

## Reading the encoder
As a non-deterministic input method, I think this is going to need to use IO pins that can trigger an interrupt, then an ISV is going to be needed to track the state of the pins.

Digging around, the ISV based approach is doable, but requires tricky logic to debounce and work out the direction. Polling the pin state in the main loop seems to be a simpler method that produces relatively reliable output. The logic is:

```C
void loop_encoder() {
  current_clk = digitalRead(RE_CLOCK_PIN);

  if ((current_clk != last_clk) && (current_clk == 1)) {
    if (digitalRead(RE_DATA_PIN) != current_clk) {
      // clockwise rotation
    } else {
      // counter-clockwise rotation
    }
  }
  last_clk = current_clk;
}
```
This requires the following setup:

```C
void initialise_encoder_pins() {
  Serial.println("Initialising encoder pins");
  pinMode(RE_CLOCK_PIN, INPUT);
  pinMode(RE_DATA_PIN, INPUT);
  last_clk = digitalRead(RE_CLOCK_PIN);
}
```

Where `RE_CLOCK_PIN` points to the pin wired to the `CLK` of the encoder, and `RE_DATA_PIN` points to the pin wired to the `DATA` of the encoder. These are sometimes labelled `A` and `B` respectively.
