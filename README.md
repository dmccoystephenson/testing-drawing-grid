# Patchwork

**Patchwork** is a lightweight tool for visualizing 2D environments, built with Python. It serves as a testbed for experimenting with graphical rendering of grids and virtual environments using various graphics libraries—starting with **Pygame**.

This project is part of the [Viron](https://github.com/Preponderous-Software/Viron) ecosystem and provides a visual layer to its simulated environments.

## Features

- Grid-based rendering of 2D environments
- Initial support for **Pygame**
- Caching of created environments in `environments.json` so a grid size can be re-loaded instead of re-created
- Modular structure designed for future support of other graphics libraries
- Clean interface for testing Viron entity placement and behavior
- **RenderWindow** class for simplified Pygame window management

## RenderWindow

Patchwork provides a `RenderWindow` class that encapsulates Pygame initialization and window management. This class is designed for **composition, not inheritance**, making it easy to integrate into projects without subclassing.

### Key Features

- Automatic Pygame initialization
- Window and surface management
- Event loop handling with custom event handlers
- Frame rate control
- Teardown via `close()` or the context-manager protocol
- Clean API for common rendering operations

### Basic Usage

```python
from render_window import RenderWindow
import pygame

# Create window
window = RenderWindow("My Application", 800, 600)
surface = window.get_surface()

# Register custom event handlers
def handle_input(event):
    if event.type == pygame.KEYDOWN:
        print(f"Key pressed: {event.key}")

window.register_event_handler(handle_input)

# Main loop
while window.should_continue():
    surface.fill((0, 0, 0))
    # ... render your content ...
    pygame.display.update()
    window.tick(60)  # 60 FPS

window.close()
```

`close()` shuts Pygame back down and marks the window as no longer running, so any
further call to `should_continue()` returns `False`. The window can also be used as a
context manager, which tears it down even if the loop body raises:

```python
with RenderWindow("My Application", 800, 600) as window:
    surface = window.get_surface()
    while window.should_continue():
        # ... render your content ...
        pygame.display.update()
        window.tick(60)
```

`main.py` uses `RenderWindow` for its own window and render loop, so it doubles as a
worked example of integrating the class with `Graphik`.

## Getting Started

### Prerequisites

- Python 3.10+
- [Pygame](https://www.pygame.org/) (`pip install pygame`)
- [Viron](https://github.com/Preponderous-Software/Viron), which is vendored as a Git submodule and is also expected to be running as a server (see below)
- Docker, if Viron is to be started from the bundled Compose file

### Setup

Clone the repository along with the `Viron` submodule:

```bash
git clone --recurse-submodules https://github.com/Preponderous-Software/patchwork.git
cd patchwork
pip install pygame
```

If the repository was already cloned without `--recurse-submodules`, the submodule can be populated afterwards:

```bash
git submodule update --init --recursive
```

The submodule is required at runtime: `main.py` imports Viron's `EnvironmentService` and `LocationService` from the `Viron/` directory.

### Starting Viron

Patchwork expects a Viron server to be reachable at `http://localhost:9999`. On Windows, the bundled batch scripts start and stop it:

```bat
up.bat
down.bat
```

The equivalent commands on other platforms are:

```bash
docker compose -f Viron/compose.yml up -d --build
docker compose -f Viron/compose.yml down --remove-orphans --volumes
```

Note that `down.bat` passes `--volumes`, so stopping Viron this way also deletes its database volumes. Any environments recorded in `environments.json` will no longer resolve afterwards.

### Running

To launch the Patchwork visualization:

```bash
python main.py
```

An optional first argument sets the grid size, which defaults to `50`. A value that cannot be parsed as an integer also falls back to `50`.

```bash
python main.py 100
```

Passing `--exit-after-create` as the second argument renders a newly created environment once and then exits after roughly two seconds, instead of entering the render loop. It has no effect when the requested grid size is already cached in `environments.json`, since no environment is created in that case. Because the flag is read positionally, a grid size must be supplied before it:

```bash
python main.py 100 --exit-after-create
```

Created environments are recorded in `environments.json`, keyed by grid count and grid size (for example `1x50`; the grid count is currently fixed at `1`). A key that is already present in that file is re-loaded from Viron rather than re-created, so the file should be deleted to force re-creation.

The render loop is capped at 60 frames per second via `RenderWindow.tick()`. Since each location is re-coloured at random on every frame, that cap is also what sets the rate at which the visualization re-randomizes.

`main.py` guards its entry point with `if __name__ == "__main__":`, so the module can be imported — by the test suite, or by another program wanting to call `main(gridSize, exitAfterCreate)` directly — without launching a window. `main()` also accepts `locationService` and `environmentService` arguments, which default to services pointed at `http://localhost:9999`.

### Batch environment creation

On Windows, `create_environments.bat` deletes `environments.json` and then invokes `python main.py <size> --exit-after-create` once per grid size, from `1` up to the maximum size given as its first argument (defaulting to `100`). Standard output is appended to `output.txt` and errors to `error_log.txt`.

```bat
create_environments.bat 25
```

### Running the tests

Unit tests live in `tests/` and use only the standard library's `unittest`. They mock Pygame and stand in for Viron's service modules with stubs registered in `sys.modules`, so no display, no running Viron server, and not even a populated `Viron/` submodule are required. Because Viron is stubbed rather than imported, the suite also runs on Python versions older than the 3.10 that `main.py` itself needs:

```bash
python -m unittest discover -s tests
```

Run the command from the repository root, so that `main.py` and `render_window.py` are importable.

## Use Cases

- Visualization of entity grids and spatial data from Viron
- Debugging simulations in real-time
- Prototyping user interfaces or tile-based systems
- Educational demos of 2D virtual environments

## Roadmap

- [ ] Add support for other graphics libraries (Tkinter, OpenGL, etc.)
- [ ] Interactive toggling of cell states
- [ ] Layered rendering and animation
- [ ] Customizable grid styling
- [ ] Real-time interaction with live Viron simulations

## 📄 License

This project is licensed under the **Preponderous Non-Commercial License (Preponderous-NC)**.  
It is free to use, modify, and self-host for **non-commercial** purposes, but **commercial use requires a separate license**.

> **Disclaimer:** *Preponderous Software is not a legal entity.*  
> All rights to works published under this license are reserved by the copyright holder, **Daniel McCoy Stephenson**.

Full license text:  
[https://github.com/Preponderous-Software/preponderous-nc-license/blob/main/LICENSE.md](https://github.com/Preponderous-Software/preponderous-nc-license/blob/main/LICENSE.md)

---

**Created by [Daniel McCoy Stephenson](https://github.com/dmccoystephenson)** as part of the Preponderous ecosystem.
