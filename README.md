# RTTTL Player for MicroPython

A lightweight, modular RTTTL (Ring Tone Text Transfer Language) library for MicroPython, designed for the **Raspberry Pi Pico W** and compatible boards.

Play classic ringtones and melodies through a buzzer or piezo speaker with support for both blocking and non-blocking playback, looping, playback styles, and a built-in melody library.

Based on the [Arduino PlayRtttl library](https://github.com/ArminJo/PlayRtttl) by Armin Joachimsmeyer.

---

## Features

- 🎵 **Non-blocking playback** — call `update()` from your main loop; the rest of your code keeps running
- 🔁 **Looping** — loop a song a fixed number of times or forever
- 🎼 **Playback styles** — Natural, Staccato, Continuous, and custom divisors
- 📦 **22 built-in melodies** — Star Wars, Nokia, Mission Impossible, Pink Panther, and more
- 🗂️ **Modular architecture** — import only what you need; easy to extend
- 🪶 **Memory-conscious** — small and tiny melody subsets for constrained devices

---

## Hardware

Connect a passive buzzer or piezo speaker between a GPIO pin and GND. No resistor is required for most piezo buzzers; a 100 Ω series resistor is recommended for electromagnetic buzzers.

```
Pico W GPIO 8  ──────┤ buzzer ├────── GND
```

---

## Installation

Copy the `rtttl/` folder to your Pico W (e.g. using [Thonny](https://thonny.org/) or `mpremote`):

```bash
mpremote cp -r rtttl/ :
```

---

## Quick Start

### Blocking playback

```python
from rtttl import PlayRtttl
from rtttl.melodies import RTTTL_MELODIES

player = PlayRtttl(pin=8)
player.play_blocking(RTTTL_MELODIES[21])  # Nokia ringtone
```

### Non-blocking playback

```python
import time
from rtttl import PlayRtttl
from rtttl.melodies import RTTTL_MELODIES

player = PlayRtttl(pin=8)
player.start(RTTTL_MELODIES[0])  # Star Wars — starts immediately

while True:
    player.update()         # advance the player state machine
    # ... your other code here ...
    time.sleep_ms(1)
```

### Callback on completion

```python
def song_finished():
    print("Done!")

player.start(RTTTL_MELODIES[5], on_complete=song_finished)
```

### Play a random melody

```python
from rtttl.melodies import RTTTL_MELODIES_TINY

player.play_random_blocking(RTTTL_MELODIES_TINY)
```

### Change playback style

```python
from rtttl import PlayRtttl, STYLE_STACCATO

player = PlayRtttl(pin=8, style_divisor=STYLE_STACCATO)
player.play_blocking(RTTTL_MELODIES[5])
```

### Loop a song

```python
# Loop using the RTTTL l= parameter (3 times)
nokia_3x = "Nokia:d=4,o=5,b=112,l=3:8e6,8d6,f#5,g#5,8c#6,8b5,d5,e5,8b5,8a5,c#5,e5,2a5"
player.play_blocking(nokia_3x)
```

---

## API Reference

### `PlayRtttl(pin, style_divisor=STYLE_NATURAL)`

| Method | Description |
|--------|-------------|
| `start(rtttl, on_complete=None)` | Begin non-blocking playback. Returns `True` on success. |
| `update()` | Advance the state machine. Call repeatedly in your main loop. Returns `True` while playing. |
| `play_blocking(rtttl)` | Play synchronously; blocks until the song finishes. |
| `stop()` | Immediately silence the buzzer and halt playback. |
| `is_playing()` | Returns `True` if playback is in progress. |
| `play_random(songs, on_complete=None)` | Start a random song from a list (non-blocking). |
| `play_random_blocking(songs)` | Play a random song from a list (blocking). |
| `set_style(style_divisor)` | Change the default playback style. |

---

## Playback Styles

| Constant | Divisor | Effect |
|----------|---------|--------|
| `STYLE_NATURAL` | 16 | Small gap between notes *(default)* |
| `STYLE_STACCATO` | 2 | Short notes, long gaps |
| `STYLE_CONTINUOUS` | 0 | Notes run end-to-end, no gap |
| `STYLE_4` | 4 | Note length − ¼ |
| `STYLE_8` | 8 | Note length − ⅛ |

The style can also be embedded in the RTTTL string itself with the `s=` parameter (`s=N`, `s=S`, `s=C`).

---

## Built-in Melodies

| Index | Name | Index | Name |
|-------|------|-------|------|
| 0 | Star Wars | 11 | Looney Tunes |
| 1 | Mahna Mahna | 12 | 20th Century Fox |
| 2 | Leisure Suit | 13 | James Bond |
| 3 | Mission Impossible | 14 | The Good, the Bad and the Ugly |
| 4 | The Entertainer | 15 | Pink Panther |
| 5 | Muppets | 16 | The A-Team |
| 6 | Flintstones | 17 | Jeopardy |
| 7 | YMCA | 18 | Gadget |
| 8 | The Simpsons | 19 | Smurfs |
| 9 | Indiana Jones | 20 | Toccata |
| 10 | Take On Me | 21 | Nokia (corrected) |

**Memory-saving subsets:**

```python
from rtttl.melodies import RTTTL_MELODIES_SMALL  # 11 melodies
from rtttl.melodies import RTTTL_MELODIES_TINY   #  6 melodies
```

---

## Module Structure

```
rtttl/
├── __init__.py      # Public re-exports
├── constants.py     # Note table, style constants, defaults
├── notes.py         # Frequency calculation, style-char conversion
├── parser.py        # RTTTL header + note-token parser
├── player.py        # PlayRtttl — PWM driver and state machine
└── melodies.py      # Built-in RTTTL strings and subsets
```

Each module can be imported independently, which is useful on memory-constrained devices where you may only need the parser or the frequency table.

---

## RTTTL Format

RTTTL strings have the format:

```
Name:d=<duration>,o=<octave>,b=<bpm>[,l=<loops>][,s=<style>]:<notes>
```

**Example:**
```
Nokia:d=4,o=5,b=112:8e6,8d6,f#5,g#5,8c#6,8b5,d5,e5,8b5,8a5,c#5,e5,2a5
```

| Parameter | Meaning |
|-----------|---------|
| `d=` | Default note duration (1, 2, 4, 8, 16, 32) |
| `o=` | Default octave (3–7) |
| `b=` | Tempo in BPM |
| `l=` | Loop count (15 = forever) |
| `s=` | Style (`N`atural, `S`taccato, `C`ontinuous) |

Notes are comma-separated tokens in the form `[duration]<note>[#][.]<octave>`.

---

## License

MIT License. See `LICENSE` for details.

The included melodies may be subject to copyright. You are responsible for ensuring your use complies with applicable laws.
