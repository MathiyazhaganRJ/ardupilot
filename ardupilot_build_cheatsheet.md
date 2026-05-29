# ArduPilot Build Cheat Sheet (WSL/Ubuntu)

This is a simplified, quick-reference guide for building ArduPilot firmware. It focuses only on the essential commands you will use day-to-day.

## 1. Initial Setup (One-time only)
These commands set up your ArduPilot environment on a fresh WSL/Ubuntu installation.

```bash
# Navigate to your Linux home directory
cd ~

# Clone the repository
git clone https://github.com/ArduPilot/ardupilot.git
cd ardupilot

# Initialize submodules (downloads required libraries)
git submodule update --init --recursive

# Install prerequisite tools and compilers
Tools/environment_install/install-prereqs-ubuntu.sh -y

# Reload your terminal profile to apply paths
. ~/.profile
```

---

## 2. Getting the Latest Code (Before building)
If you haven't built in a while, it's good practice to fetch the latest updates from the ArduPilot servers. Run these from inside your `~/ardupilot` directory.

```bash
# Pull the latest changes from the master branch
git pull

# Update submodules in case dependencies changed
git submodule update --init --recursive
```

---

## 3. Configuring the Build
Before you compile, you must tell the `waf` build system which flight controller hardware you are using.

```bash
# Standard configuration for a specific board
./waf configure --board <board_name>

# Example: Matek F405-TE
./waf configure --board MatekF405-TE

# Example: Cube Orange
./waf configure --board CubeOrange

# List all supported boards if you don't know the exact name
./waf list_boards
```

---

## 4. Compiling the Firmware
Once configured, compile the code for your specific vehicle type. 

```bash
# Build for Multirotors / Helicopters
./waf copter

# Build for Fixed Wing
./waf plane

# Build for Rovers / Boats
./waf rover
```

### Where is the compiled firmware?
Once finished, the compiled files are located at:
`~/ardupilot/build/<board_name>/bin/`

Look for the file ending in `.apj` (e.g., `arduplane.apj` or `arducopter.apj`). This is the file you will flash to your flight controller using Mission Planner.

---

## 5. Switching Between Hardware and SITL Simulation
When using the same ArduPilot repository to build firmware AND run simulations, you must remember that **running a simulation changes the build configuration**.

### Running SITL (Simulation)
When simulating, you do not use `./waf` directly. The simulation script configures everything for you.
```bash
# This automatically configures the codebase for SITL and launches JSBSim
Tools/autotest/sim_vehicle.py -v ArduPlane -f jsbsim:Rascal --console --map
```

### Switching Back to Hardware
If you ran SITL and now want to build firmware for your real flight controller, you **must** reconfigure the board, otherwise it will try to build a PC simulation!
```bash
# 1. Reconfigure for your specific board
./waf configure --board MatekF405-TE

# 2. Compile the firmware
./waf plane
```

---

## 6. Helpful WAF Commands

If you run into weird build errors, or if you change branches, it's highly recommended to clean your build environment and start fresh.

```bash
# Cleans out the current build files (fast)
./waf clean

# Completely nukes the build directory and configuration (very thorough)
./waf distclean

# Clean and reconfigure (recommended if you switch between branches)
./waf distclean
./waf configure --board <board_name>
```

> [!TIP]
> **Windows File Explorer Access:**
> To easily grab your `.apj` firmware files and drag them into Mission Planner, open your Windows File Explorer, click the address bar at the top, and type `\\wsl$\Ubuntu\home\`. You can navigate directly to your `ardupilot/build/` folder from there!
