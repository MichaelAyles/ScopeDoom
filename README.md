# ScopeDoom

**DOOM rendered on a hardware oscilloscope via sound card**

![ScopeDoom Demo](assets/scope_20251125_134832_678.bmp)

## Overview

ScopeDoom renders DOOM's 3D world as wireframe vector graphics on a real oscilloscope. It uses your computer's sound card as a dual-channel DAC - the left channel drives the X-axis, the right channel drives the Y-axis, and the scope displays in X-Y mode.

## How It Works

```
DOOM Engine (C)
    |
    | Extract walls & entities as vectors
    v
Unix Domain Socket
    |
    | JSON: {walls: [...], entities: [...]}
    v
doom_scope.py (Python)
    |
    | Convert to X-Y point stream
    v
Sound Card (44.1kHz stereo)
    |
    | Left = X, Right = Y
    v
Oscilloscope (X-Y Mode)
```

### Key Components

- **doom_scope.py** - Main renderer that receives DOOM vectors and outputs audio
- **scope_capture.py** - Capture oscilloscope screenshots via VXI-11 (Siglent scopes)
- **scope_output.py** - Test patterns (squares, circles) for scope calibration
- **doom/source/** - Modified DOOM engine with vector extraction

## Hardware Setup

### What You Need

- Computer with 3.5mm audio output (or USB audio interface)
- Oscilloscope with X-Y mode
- Two load resistors (1k ohm recommended)
- Audio cable (3.5mm to bare wires or RCA)

### Wiring

```
MacBook Pro 3.5mm Jack
        |
        +--- Left (Tip) ----[1k]----+---- CH1 (X)
        |                           |
        +--- Right (Ring) --[1k]----+---- CH2 (Y)
        |                           |
        +--- Ground (Sleeve) -------+---- GND
```

### Scope Settings

- **Mode:** X-Y
- **Channel 1:** X input (left audio)
- **Channel 2:** Y input (right audio)
- **Scale:** Start at 500mV/div, adjust as needed
- **Coupling:** DC or AC (AC removes DC offset from audio)

## Quick Start

### 1. Test Your Setup

```bash
# Generate test patterns (no DOOM needed)
python3 scope_wav_test.py
afplay scope_square.wav  # Should show a square on scope
```

### 2. Run with DOOM

```bash
# Terminal 1: Start the scope renderer
python3 doom_scope.py

# Terminal 2: Launch DOOM (from doomgeneric build)
./doomgeneric_kicad -w 1 1
```

### 3. Play!

- Controls work in the SDL window
- Scope displays wireframe view in real-time
- WASD to move, arrows to turn, Ctrl to fire

## Building DOOM

ScopeDoom requires a modified DOOM engine that extracts vector data. See `doom/source/build.sh` for build instructions.

The key modification is extracting wall segments (`drawsegs[]`) and sprite positions (`vissprites[]`) and sending them over a Unix socket as JSON.

## Performance

| Metric | Value |
|--------|-------|
| Sample Rate | 44,100 Hz |
| Points per Frame | ~5,000-10,000 |
| Effective Refresh | 4-8 Hz |
| Walls Rendered | 30-50 typical |
| Entities Rendered | 0-10 |

The refresh rate depends on scene complexity. More walls/entities = more points = slower refresh. But it's still playable!

## Dependencies

```bash
pip install sounddevice numpy
```

Optional (for scope screenshots):
```bash
pip install python-vxi11
```

## Files

```
ScopeDoom/
├── doom_scope.py      # Main DOOM-to-scope renderer
├── scope_capture.py   # Oscilloscope screenshot capture
├── scope_output.py    # Test pattern generator
├── scope_wav_test.py  # WAV file test patterns
├── assets/            # Screenshots and demos
└── doom/source/       # Modified DOOM engine source
```

## Known Issues

- **DC Offset** - Mac audio has DC bias, image may not be centered
- **Visible Retrace** - No Z-axis blanking, beam movement visible
- **Aliasing** - Limited sample rate causes stepping on diagonals

## Future Ideas

- Z-axis blanking with 3-channel audio interface
- Higher sample rate (96kHz/192kHz audio)
- Optimize point ordering to minimize retrace
- Add simple HUD elements

## Credits

- DOOM by id Software
- [doomgeneric](https://github.com/ozkl/doomgeneric) portable DOOM
- Inspired by oscilloscope music artists and Vectrex DOOM

## License

GPL (inherits from DOOM source code)
