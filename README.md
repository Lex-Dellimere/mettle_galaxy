# mettle_galaxy

I wrote a simulation of two galaxies colliding. The gravity and the drawing both run as [Mettle](https://github.com/The-Mettle-Project/Mettle) kernels on an NVIDIA GPU, and [Window](https://github.com/Lex-Dellimere/Window), my window library, shows the picture.

Every star pulls on every other star. The gravity kernel works through the stars in tiles of 256. A block of 256 threads loads a tile into workgroup memory, each thread adds the pull of those stars on its own two stars, and then the block moves on to the next tile. On my RTX 4070 laptop a step for 32,768 stars takes about 2.9 ms.

The galaxies start on a parabolic orbit that brings them within about 8 units of each other. It's the setup Toomre and Toomre used in 1972 to explain long tidal tails. Each galaxy has a black hole in the middle that pulls softly inside one unit, so stars passing close don't get flung away.

Drawing is four more kernels. Each star adds its light to the four pixels around where it lands, a quarter size copy gets blurred for the glow, and a tone curve turns light into colour without clipping the bright cores. Only the finished picture comes back to the CPU, and it copies back while the GPU works on the next frame.

## Build and run

You'll need the Mettle compiler, mbuild and an NVIDIA GPU with its driver. mbuild fetches Window from the registry. The program talks to the CUDA driver directly, so you don't need the CUDA toolkit.

```bash
mbuild run galaxy
```

```bash
mbuild run galaxy -- --stars 65536
```

- `--stars N` sets how many stars, 32,768 by default.
- `--headless FILE` runs without a window and saves the last frame as a PPM. `--steps N` sets how many steps, and `--record DIR` with `--every N` saves a frame every N steps.
- `--bench` times the gravity step at 16,384 to 131,072 stars.
- `--seconds N` closes the window after N seconds.

```bash
mbuild run check
```

The check compares the GPU with the CPU. The pull on each of 4,098 stars has to match a float64 sum, and the total energy has to stay within one part in a hundred over 3,000 steps.

## Controls

Drag to turn around the galaxies and scroll to zoom. Space pauses, R starts again, [ and ] change how many steps run each frame, A turns the slow spin on and off and Escape quits.

## License

Apache-2.0
