# mettle_galaxy

Two galaxies colliding, written in [Mettle](https://github.com/The-Mettle-Project/Mettle). The gravity and the drawing both run in Mettle kernels on an NVIDIA GPU, with no shaders and no OpenGL, and [Window](https://github.com/Lex-Dellimere/Window) shows the finished picture.

Every star pulls on every other star. The gravity kernel works in tiles: a block of 256 threads loads 256 stars into workgroup memory, each thread adds their pull on its own two stars, and the block moves on to the next 256. On an RTX 4070 laptop that is about 355 billion pairs of stars a second, 2.8 ms a step for 32,768 stars and 12.1 ms for 65,536.

The galaxies start on a parabolic orbit that brings them within about 8 units of each other, with the bigger one spinning the same way it orbits, the setup Toomre and Toomre used in 1972 to explain long tidal tails. Each has a black hole at its centre that pulls softly inside one unit, so stars that pass close are not flung away.

The drawing is four more kernels: each star adds its light to the four pixels around where it lands, a quarter-size copy of the light is blurred for the glow, and a tone curve turns light into colour without clipping the bright cores. Only the finished picture comes back to the CPU.

## Build and run

You need the Mettle compiler, mbuild and an NVIDIA GPU with its driver. mbuild fetches Window from the registry. The program finds the GPU through the CUDA driver; it does not need the CUDA toolkit.

```bash
mbuild run galaxy
```

```bash
mbuild run galaxy -- --stars 65536
```

- `--stars N` sets how many stars, 32,768 by default.
- `--headless FILE` runs with no window and saves the last frame as a PPM. `--steps N` sets how many steps, and `--record DIR` with `--every N` saves a frame every N steps.
- `--bench` times the gravity step at 16,384 to 131,072 stars.
- `--seconds N` closes the window after N seconds.

```bash
mbuild run check
```

checks the GPU against the CPU: the pull on every one of 4,098 stars matches a sum in float64, and the total energy stays within one part in a hundred over 3,000 steps.

## Controls

Drag to turn around the galaxies, scroll to zoom, Space pauses, R starts again, [ and ] change how many steps run each frame, A turns the slow spin on and off, Escape quits.

## License

Apache-2.0
