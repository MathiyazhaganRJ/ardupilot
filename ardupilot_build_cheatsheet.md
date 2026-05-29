# ArduPilot Build Cheat Sheet (WSL/Ubuntu)

This is a simplified, quick-reference guide for building ArduPilot firmware and running SITL simulations.

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
If you haven't built in a while, fetch the latest updates from the ArduPilot servers.

```bash
cd ~/ardupilot
git pull
git submodule update --init --recursive
```

---

## 3. Configuring and Compiling for Hardware
Before you compile for a physical flight controller, you must configure `waf` for that specific board.

```bash
cd ~/ardupilot

# 1. Configure for your specific board (Example: Matek F405-TE)
./waf configure --board MatekF405-TE

# 2. Compile the firmware for your vehicle type
./waf plane     # For Fixed Wing
# ./waf copter  # For Multirotors
# ./waf rover   # For Rovers
```

### Where is the compiled firmware?
Once finished, the compiled files are located at:
`~/ardupilot/build/<board_name>/bin/`

Look for the `.apj` file and flash it to your flight controller using Mission Planner.

---

## 4. JSBSim Installation (For Advanced SITL)
ArduPilot has a basic simulator, but if you want high-fidelity aerodynamics for ArduPlane, you must install JSBSim.

```bash
# 1. Clone and build JSBSim in your home directory
cd ~
git clone https://github.com/JSBSim-Team/jsbsim.git
cd jsbsim
mkdir build
cd build
cmake ..
make -j4

# 2. Add JSBSim to your terminal path
nano ~/.profile
# Paste this at the very bottom: export PATH=$PATH:$HOME/jsbsim/build/src
# Save (Ctrl+O, Enter) and Exit (Ctrl+X)

# 3. Reload your terminal profile
. ~/.profile
```

---

## 5. Running SITL (Simulation)
When simulating, you do not use `./waf` directly. The simulation script configures everything for you and launches the simulator.

```bash
cd ~/ardupilot/ArduPlane

# Standard SITL (Basic physics)
../Tools/autotest/sim_vehicle.py -v ArduPlane --console --map

# JSBSim SITL (Advanced physics, requires Step 4)
# The 'Rascal' is a default model provided by ArduPilot
../Tools/autotest/sim_vehicle.py -v ArduPlane -f jsbsim:Rascal --console --map
```

> [!WARNING]
> **Switching Back to Hardware!**
> Running `sim_vehicle.py` automatically reconfigures your build environment for a PC simulation. If you want to compile firmware for your Matek board again, you **MUST** run `./waf configure --board MatekF405-TE` before running `./waf plane`.

---

## 6. Helpful WAF Commands

```bash
# Cleans out the current build files (fast)
./waf clean

# Clean and reconfigure (recommended if you switch between branches)
./waf distclean
./waf configure --board <board_name>
```
