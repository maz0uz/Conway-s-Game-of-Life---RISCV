# Conway's Game of Life - RISC-V Assembly

## Setup

### RARS Configuration

**Bitmap Display:**

- Tools > Bitmap Display
- Unit Width: 4 pixels
- Unit Height: 4 pixels
- Display Width: 512 pixels
- Display Height: 512 pixels
- Base address: 0x10040000
- Connect to MIPS

**Hex Keyboard:**

- Tools > Keyboard and Display MMIO Simulator
- Connect to MIPS

### Running the game

1. Open the .asm file in RARS
2. Hit F3 to assemble
3. Press 0, 1, 2, or 3 on the hex keyboard to pick a pattern
4. Hit F5 to run
5. Watch the bitmap display update for 1000 generations

## Initial Patterns

**Pattern 0 - Glider (Key 0)**  
Small 3x3 pattern that moves diagonally across the screen.

**Pattern 1 - Glider Gun (Key 1)**  
A large, complex stationary structure that acts as a factory, constantly creating and shooting out new Gliders

**Pattern 2 - Pulsar (Key 2)**  
A large, beautiful period-3 oscillator. It looks like a star that rhythmically explodes and implodes.

**Pattern 3 - 8-Cell Line (Key 3)**  
A simple straight horizontal line of 8 alive cells. Despite looking simple, it evolves into a very complex, chaotic pattern that lasts for many generations.

## Implementation Details/Code Structure

**Grid Storage:**

- 1D arrays in row-major order
- Each cell is 4 bytes (word) storing the color value
- Total size per grid: 128 _ 128 _ 4 = 65536 bytes

**Seed Loading:**

- Seeds are defined as variable-length coordinate lists (pairs of X, Y bytes) terminated by a sentinel value (0x80).
- Loaded relative to the center of the screen (Row 62, Column 62) so patterns have room to expand in all directions.
- Generated using a loop that reads each coordinate pair and calculates the specific memory address offset for that cell.

**Double Buffering:**

- `currentState` holds the generation being evaluated
- `nextState` holds the generation being computed
- After computation, `copyState` copies `nextState` to `currentState` and display
- This prevents reading partially updated data

## Problems We Ran Into

**Display Memory Address Conflict:**
We initially set the bitmap display to use address 0x10040000, but later discovered RARS was mapping the display to a different memory region. This mismatch caused the program to stop working completely even though all our game logic was correct. We had to verify the exact base address through RARS documentation and testing.

**Uninitialized Generation Counter:**
The loop controlling how many generations to run wasn't properly initialized, causing the simulation to either run indefinitely or not run at all. This oversight meant that even with correct display addressing and game logic, the simulation wouldn't progress through generations as expected.

**Static Data Memory Corruption:**
We initially stored our game grids (currentState and nextState) in the static data section, which sometimes led to memory corruption during extended runs. The large 128x128 grids (each 65536 bytes) were competing with other data in limited memory space. We learned that we should have allocated this memory dynamically from the heap to ensure proper memory isolation and prevent corruption during cache analysis experiments.

## Limitations

The simulation runs slowly, which is problematic when collecting large amounts of data for cache analysis.
