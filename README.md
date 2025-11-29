# IsaacSim-Pegasus-Ardupilot Check

## Installation Script

```bash
# --- 1. Isaac Sim installation (Ubuntu 22.04) ---

# Go to the home directory
cd ~

# Create a new directory to store the Isaac Sim installation
mkdir -p IsaacSim
cd IsaacSim

# Download Isaac Sim Standalone 5.1.0
wget https://download.isaacsim.omniverse.nvidia.com/isaac-sim-standalone-5.1.0-linux-x86_64.zip

# Unzip
mv ~/Downloads/isaac-sim-standalone-5.1.0-linux-x86_64.zip .
unzip isaac-sim-standalone-5.1.0-linux-x86_64.zip

# Post-install scripts
./post_install.sh
./isaac-sim.selector.sh

# Delete zip
rm isaac-sim-standalone-5.1.0-linux-x86_64.zip


# --- 2. Ardupilot installation ---

# Install dependencies
sudo apt install git make cmake python3-pip build-essential ccache g++ gawk wget valgrind screen python3-pexpect pkg-config libtool libxml2-dev libxslt1-dev xterm
sudo apt-get install python3-wxgtk4.0 -y --no-install-recommends
pip install pymavlink MAVProxy kconfiglib jinja2 empy jsonschema pyros-genmsg packaging toml numpy future future lxml pymavlink pyserial geocoder empy==3.3.4 ptyprocess dronecan flake8 junitparser pygame intelhex --user

# Clone ArduPilot
cd ~
git clone https://github.com/ArduPilot/ardupilot.git
cd ardupilot
git checkout ArduCopter-stable
git submodule update --init --recursive

# Install the ArduPilot toolchain
Tools/environment_install/install-prereqs-ubuntu.sh -y
. ~/.profile


# Test SITL (builds automatically)
Tools/autotest/sim_vehicle.py -v ArduCopter --console --map


# --- 3. Pegasus installation ---

# Option 1: With HTTPS
git clone https://github.com/PegasusSimulator/PegasusSimulator.git

# Install pymavlink MAVProxy
# Don`t upgrade PIP!!!
~/IsaacSim/python.sh -m pip install pymavlink MAVProxy

#install xterm
sudo apt install xterm -y

#install pyzmq opencv
~/IsaacSim/python.sh -m pip install pyzmq opencv-python-headless

#replace 11_ardupilot_raycast_spawn_fixed.py in ~/PegasusSimulator/examples from thiss repository
#replace ardupilot_mavlink_backend.py in ~/PegasusSimulator/extensions/pegasus.simulator/pegasus/simulator/logic/backends
#add to the end of your
nano ~/.bashrc

# -------------------------------------------------------------------
# Isaac Sim configuration
# -------------------------------------------------------------------
export ISAACSIM_PATH="${HOME}/IsaacSim"
export ISAACSIM_PYTHON="${ISAACSIM_PATH}/python.sh"
export ISAACSIM="${ISAACSIM_PATH}/isaac-sim.sh"
export PYTHONPATH="$PYTHONPATH:$HOME/PegasusSimulator/extensions/pegasus.simulator"


isaac_run() {
    if [ ! -x "$ISAACSIM_PYTHON" ]; then
        echo "Error: IsaacSim python.sh not found at: $ISAACSIM_PYTHON"
        return 1
    fi
    if [ ! -x "$ISAACSIM" ]; then
        echo "Error: IsaacSim launcher not found at: $ISAACSIM"
        return 1
    fi

    if [ $# -eq 0 ]; then
        echo "Launching Isaac Sim GUI..."
        "${ISAACSIM}"
        return 0
    fi

    if [[ "$1" == --* ]]; then
        echo "Launching Isaac Sim with flags: $@"
        "${ISAACSIM}" "$@"
        return 0
    fi

    if [ -f "$1" ]; then
        local SCRIPT_PATH="$1"
        shift
        echo "Running Python script with Isaac Sim Python:"
        echo "    $SCRIPT_PATH"
        "${ISAACSIM_PYTHON}" "$SCRIPT_PATH" "$@"
        return 0
    fi

    echo "Unknown argument or file not found: '$1'"
    echo "Usage:"
    echo "  isaac_run                 → launch Isaac Sim GUI"
    echo "  isaac_run my_script.py    → run script using IsaacSim Python"
    echo "  isaac_run --headless      → run in headless mode"
    return 1
}
source $HOME/ardupilot/Tools/completion/completion.bash


#run example
isaac_run ~/PegasusSimulator/examples/11_ardupilot_multi_vehicle.py

#wait and then try
mode guided
arm throttle
takeoff 3
