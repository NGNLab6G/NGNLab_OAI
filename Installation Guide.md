# OAI FlexRIC 5G Testbed Setup Guide

<details>
<summary>Section 1: System Preparation and Prerequisites</summary>
  
```bash
# Update system packages
sudo apt update -y
```
Explanation: Fetches the latest list of available software packages from repositories.

```bash
# Upgrade system packages
sudo apt upgrade -y
```
Explanation: Installs the latest versions of all installed packages.

```bash
# Install build tools
sudo apt install -y build-essential
```
Explanation: Installs essential development tools like GCC (compiler), G++ (C++ compiler), Make, and other utilities.

```bash
# Install GCC 13
sudo apt install -y gcc-13 g++-13 cpp-13
```
Explanation: Installs version 13 of the GNU Compiler Collection (GCC) and related tools.

```bash
# Configure alternatives for gcc and g++
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-13 100 --slave /usr/bin/g++ g++ /usr/bin/g++-13 --slave /usr/bin/gcov gcov /usr/bin/gcov-13
sudo update-alternatives --config gcc
```
Explanation: Set GCC-13 as the default compiler when you run gcc or g++.

```bash
# Verify version
gcc --version
g++ --version
```
Explanation: Show the current version of GCC/G++.

```bash
# Install Wireshark and tcpdump
sudo apt install -y wireshark tcpdump
```
Explanation: Install network packet capture tools.

```bash
# Install dependencies
sudo apt install libsctp-dev cmake-curses-gui libpcre2-dev
```
Explanation:  
libsctp-dev: Adds support for SCTP protocol, which is used for certain signaling messages.  
cmake-curses-gui: Provides a terminal-based configuration tool to help set up build options.  
libpcre2-dev: Enables regular expression support needed by some OAI modules.  

```bash
# Build and install SWIG
git clone https://github.com/swig/swig.git
cd swig
git checkout release-4.1
./autogen.sh
./configure --prefix=/usr/
make -j8
sudo make install
swig -version
```
Explanation: Download, prepare, compile, and install SWIG, a tool that generates language bindings, enabling C++ code to interface with Python and other languages.

```bash
# Install Python headers
sudo apt install python3-dev
```
Explanation: Installs development files for Python.
</details>

<details>
<summary>Section 2: Clone and Build OAI with E2 Support and FlexRIC</summary>

```bash
# Clone the OAI repository
cd ~/OAI
git clone https://gitlab.eurecom.fr/oai/openairinterface5g
```
Explanation: Moves to your workspace and downloads the full OAI code from EURECOM’s GitLab.

```bash
# Navigate to build scripts
cd openairinterface5g/cmake_targets/
```
Explanation: Changes directory to where build configurations are located.

```bash
# First build for setup
./build_oai -I
```
Explanation: Runs initial build scripts that check dependencies, prepare the environment, and verify compatibility.

```bash
# Enable E2 support and FlexRIC
./build_oai --gNB --nrUE --build-e2 --cmake-opt -DE2AP_VERSION=E2AP_V2 --cmake-opt -DKPM_VERSION=KPM_V2_03 --ninja
```
Explanation: Builds the gNB and UE with E2 interface support, using Ninja for faster compilation.

```bash
# Pull FlexRIC submodule
cd openairinterface5g/openair2/E2AP/flexric
git submodule init && git submodule update
```
Explanation: Downloads the FlexRIC version compatible with your OAI code.

```bash
# Build FlexRIC with multi-language support
mkdir build && cd build && cmake -DXAPP_MULTILANGUAGE=ON .. && make -j8
```
Explanation: Creates a build directory, configures build system with multi-language support, and compiles FlexRIC.

```bash
sudo make install
```
Explanation: Installs FlexRIC system-wide.

```bash
ctest -j8 --output-on-failure
```
Explanation: Runs tests to verify correct installation.
</details>

<details>
<summary>Section 3: Setup OAI 5G Core Using Docker</summary>

```bash
# Install Docker and its dependencies
sudo apt install -y git net-tools putty ca-certificates curl
```
Explanation: Installs essential tools for repo management, network configuration, SSH access, secure connections, and downloading.

```bash
# Setup Docker GPG key and repository
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
```
Explanation: Download and store the Docker GPG key for verified package installation.

```bash
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
Explanation: Adds Docker's official repo to your system sources.

```bash
# Install Docker components
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
Explanation: Installs Docker Engine, CLI tools, container runtime, build plugin, and compose plugin for managing containers.

```bash
# Enable Docker without sudo
sudo usermod -a -G docker $(whoami)
reboot
```
Explanation: Adds your user to Docker’s group, so Docker commands don't need root permissions, then restarts to apply changes.

```bash
# Download and run the OAI core containers
wget -O ~/oai-cn5g.zip "https://gitlab.eurecom.fr/oai/openairinterface5g/-/archive/develop/openairinterface5g-develop.zip?path=doc/tutorial_resources/oai-cn5g"
unzip ~/oai-cn5g.zip
mv ~/OAI/openairinterface5g-develop-doc-tutorial_resources-oai-cn5g/doc/tutorial_resources/oai-cn5g ~/OAI/oai-cn5g
rm -r ~/openairinterface5g-develop-doc-tutorial_resources-oai-cn5g ~/oai-cn5g.zip
cd ~/OAI/oai-cn5g
docker compose pull
docker compose up -d
```
Explanation: Download, extract, and run OAI 5G core containers.

```bash
# Verify Docker containers status
docker compose -f docker-compose.yaml ps -a
```
Explanation: Lists all containers related to the specified Compose project, including both running and stopped containers.
</details>

<details>
<summary>Section 4: Start nearRT-RIC and OAI RAN with E2 Agent</summary>

```bash
# Start nearRT-RIC
cd ~/OAI/openairinterface5g/openair2/E2AP/flexric/build/examples/ric
./nearRT-RIC
```
Explanation: Navigates to the RIC example directory and launches the near Real-Time RAN Intelligent Controller (nearRT-RIC).

```bash
# Start OAI RAN softmodem with E2 agent enabled
cd ~/OAI/openairinterface5g/cmake_targets/ran_build/build
sudo ./nr-softmodem \
  -O ~/OAI/openairinterface5g/targets/PROJECTS/GENERIC-NR-5GC/CONF/gnb.sa.band78.fr1.106PRB.usrpb210.conf \
  --rfsim \
  --e2_agent.near_ric_ip_addr 127.0.0.1 \
  --e2_agent.sm_dir /usr/local/lib/flexric/ \
  --gNBs.[0].min_rxtxtime 6
```
Explanation: Starts the 5G base station emulator (gNB) with radio frequency simulation and E2 agent enabled, which connects to the nearRT-RIC.  
-O: Specifies the configuration file for gNB parameters.  
--rfsim: Simulates radio functions without hardware.  
--e2_agent.near_ric_ip_addr: IP of the nearRT-RIC for E2 communication.  
--e2_agent.sm_dir: Directory containing Service Models for E2 interface.  
--gNBs.[0].min_rxtxtime: Sets minimum RX/TX timing.  
</details>

<details>
<summary>Section 5: Synchronization Setup</summary>

```bash
# Update package lists
sudo apt-get update
```
Explanation: Refreshes the list of available packages.

```bash
# Install Linux PTP package
sudo apt-get install linuxptp
```
Explanation: Installs the Precision Time Protocol (PTP) software.

```bash
# Check hardware timestamping support
sudo ethtool -T eno1
```
Explanation: Queries network interface eno1 to check if hardware timestamping is available.

```bash
# Start PTP daemon on interface
sudo ptp4l -m -i eno1
```
Explanation: Runs Precision Time Protocol daemon with monitoring (-m) on interface eno1.

```bash
# Synchronize system clock with PTP clock hardware
sudo phc2sys -s /dev/ptp1 -c CLOCK_REALTIME -w -m -x
```
Explanation: Synchronizes the system clock (CLOCK_REALTIME) to the PTP hardware clock /dev/ptp1.
</details>

<details>
<summary>Section 6: Start Three UEs Using Scripts</summary>

```bash
# Prepare and verify the multi-UE script
cd ~/OAI/openairinterface5g/tools/scripts
ls -l multi-ue.sh
chmod +x multi-ue.sh
```
Explanation: Navigate to scripts, check script permissions, and make the multi-UE script executable.

```bash
# Launch UE1
sudo ~/OAI/openairinterface5g/tools/scripts/multi-ue.sh -c1
sudo ~/OAI/openairinterface5g/tools/scripts/multi-ue.sh -o1
```
```bash
sudo ./nr-uesoftmodem \
  -r 106 --numerology 1 --band 78 -C 3619200000 \
  --rfsim \
  --uicc0.imsi 001010000000001 \
  --rfsimulator.serveraddr 10.201.1.100
```
Explanation: To simulate a UE with distinct IMSI and RF simulator IP.  
-c1: Sets up an isolated environment and network namespace for UE1.  
-o1: Starts logging and captures the output for UE1.  
nr-uesoftmodem command launches the UE emulator with specific radio and network parameters:  
-r 106: Configures the number of resource blocks for the bandwidth allocation.  
--numerology 1: Specifies a subcarrier spacing of 15 kHz as defined in 5G NR standards.  
--band 78: Selects frequency band 78 used for 5G NR operation.  
-C 3619200000: Sets the carrier frequency to 3.6192 GHz.  
--rfsim: Enables simulated radio frequency operation without actual hardware.  
--uicc0.imsi 001010000000001: Assigns a unique IMSI (International Mobile Subscriber Identity) to UE1, identifying it distinctively in the network.  
--rfsimulator.serveraddr 10.201.1.100: Defines the IP address of the RF simulator server that UE1 communicates with for its radio environment.  

Use similar commands updating the IMSI and RF simulator IP accordingly for each UE.

```bash
# Launch UE2
sudo ~/OAI/openairinterface5g/tools/scripts/multi-ue.sh -c2
sudo ~/OAI/openairinterface5g/tools/scripts/multi-ue.sh -o2
```
```bash
sudo ./nr-uesoftmodem \
  -r 106 --numerology 1 --band 78 -C 3619200000 \
  --rfsim \
  --uicc0.imsi 001010000000002 \
  --rfsimulator.serveraddr 10.202.1.100
```

```bash
# Launch UE3
sudo ~/OAI/openairinterface5g/tools/scripts/multi-ue.sh -c3
sudo ~/OAI/openairinterface5g/tools/scripts/multi-ue.sh -o3
```
```bash
sudo ./nr-uesoftmodem \
  -r 106 --numerology 1 --band 78 -C 3619200000 \
  --rfsim \
  --uicc0.imsi 001010000000003 \
  --rfsimulator.serveraddr 10.203.1.100
```
Check gNB Logs for Successful UE Connection.
</details>

<details>
<summary>Section 7: Docker Bridge Network Update and System Setup</summary>

```bash
# Inspect current Docker bridge network
docker network inspect bridge
```
Explanation: Shows configuration of the default Docker network bridge.

```bash
# Modify Docker bridge subnet
sudo nano /etc/docker/daemon.json
```

```bash
# Add
{
  "bip": "192.168.90.1/24"
}
```
Explanation: Sets Docker bridge IP network manually. Prevents IP overlap that can cause network issues between containers and host.

```bash
# Restart Docker
docker compose down
sudo systemctl restart docker
```

```bash
# Verify the new Docker bridge network
docker network inspect bridge
```

```bash
# Restart containers
docker compose up -d
```
Explanation: Bring Docker containers down, restart Docker daemon for changes to take effect, verify new network, and restart containers.

```bash
# Enable IP forwarding and configure packet forwarding
sudo sysctl -w net.ipv4.conf.all.forwarding=1
sudo iptables -P FORWARD ACCEPT
sudo sysctl -w net.ipv4.ip_forward=1
```
Explanation: Enable system-wide IPv4 forwarding and set iptables to accept forwarding packets.
</details>

<details>
<summary>Section 8: Setup Networking for UEs and iperf3 Testing</summary>

```bash
# Install iperf3 inside Docker DN container
docker exec -it oai-ext-dn bash
apt-get update && apt-get install -y iperf3
```
Explanation: Enters the Data Network (DN) container and installs iperf3 for network throughput testing. iperf3 generates and measures traffic to simulate real data flows.

```bash
# Verify IP addresses on DN and UE network interfaces
docker exec -it oai-ext-dn ip addr show eth0
sudo ip netns exec ue1 ip addr show v-ue1
sudo ip netns exec ue2 ip addr show v-ue2
sudo ip netns exec ue3 ip addr show v-ue3
```
Explanation: Check IP assignment on the DN container and UE namespaces.

```bash
# Assign IPs and bring up host-side veth interfaces
sudo ip addr add 10.201.1.254/24 dev v-eth1
sudo ip addr add 10.202.1.254/24 dev v-eth2
sudo ip addr add 10.203.1.254/24 dev v-eth3
sudo ip link set v-eth1 up
sudo ip link set v-eth2 up
sudo ip link set v-eth3 up
```
Explanation: Configure virtual Ethernet interfaces on host with IPs and activate them. These connect host and UE namespaces enabling network traffic flow.

```bash
# Add default routes inside UE namespaces
sudo ip netns exec ue1 ip route add default via 10.201.1.254 dev v-ue1
sudo ip netns exec ue2 ip route add default via 10.202.1.254 dev v-ue2
sudo ip netns exec ue3 ip route add default via 10.203.1.254 dev v-ue3
```
Explanation: Set the gateway for UE traffic to host-side veth IPs. Allows UEs to send traffic outside their namespace.

```bash
# Setup NAT for UE subnets to Docker subnet
sudo iptables -t nat -A POSTROUTING -s 10.201.1.0/24 -d 192.168.70.128/26 -j MASQUERADE
sudo iptables -t nat -A POSTROUTING -s 10.202.1.0/24 -d 192.168.70.128/26 -j MASQUERADE
sudo iptables -t nat -A POSTROUTING -s 10.203.1.0/24 -d 192.168.70.128/26 -j MASQUERADE
```
Explanation: Use masquerading to translate UE subnet IP addresses when accessing the Docker subnet.

```bash
# Update firewall forwarding rules for traffic flow
sudo iptables -A FORWARD -i v-eth1 -o oai-cn5g -j ACCEPT
sudo iptables -A FORWARD -i v-eth2 -o oai-cn5g -j ACCEPT
sudo iptables -A FORWARD -i v-eth3 -o oai-cn5g -j ACCEPT
sudo iptables -A FORWARD -i oai-cn5g -o v-eth1 -m state --state ESTABLISHED,RELATED -j ACCEPT
sudo iptables -A FORWARD -i oai-cn5g -o v-eth2 -m state --state ESTABLISHED,RELATED -j ACCEPT
sudo iptables -A FORWARD -i oai-cn5g -o v-eth3 -m state --state ESTABLISHED,RELATED -j ACCEPT
```
Explanation: Allow forwarding of packets between veth interfaces and Docker containers for inbound and established connections. Supports proper bidirectional network communication.

```bash
# Ping tests to verify network connectivity
sudo ip netns exec ue1 ping -c 3 10.201.1.254
sudo ip netns exec ue2 ping -c 3 10.202.1.254
sudo ip netns exec ue3 ping -c 3 10.203.1.254
sudo ip netns exec ue1 ping -c 3 192.168.70.134
sudo ip netns exec ue2 ping -c 3 192.168.70.134
sudo ip netns exec ue3 ping -c 3 192.168.70.134
```
Explanation: Pings between UEs and host-side interfaces, then towards the UPF IP.
</details>

<details>
<summary>Section 9: Generate Traffic per UE using iperf3</summary>

```bash
# Run iperf3 servers inside UE namespaces
sudo ip netns exec ue1 iperf3 -s -p 5201
sudo ip netns exec ue2 iperf3 -s -p 5202
sudo ip netns exec ue3 iperf3 -s -p 5203
```
Explanation: Start iperf3 servers on different ports inside each UE's network namespace. To receive traffic for throughput testing.

```bash
# Run iperf3 clients from DN container for downlink tests
docker exec -it oai-ext-dn iperf3 -c 10.201.1.1 -p 5201 -t 60 -i 1 -R
docker exec -it oai-ext-dn iperf3 -c 10.202.1.2 -p 5202 -t 60 -i 1 -R
docker exec -it oai-ext-dn iperf3 -c 10.203.1.3 -p 5203 -t 60 -i 1 -R
```
Explanation: From the Data Network container, run iperf3 clients connecting to each UE’s server to generate downlink traffic for 60 seconds. Tests actual data flow and network performance in the testbed.
</details>

<details>
<summary>Section 10: Data collection</summary>

```bash
# Install SQLite browser
sudo apt install sqlitebrowser
```
Explanation: Installs a graphical tool to manage SQLite databases.

```bash
# Create and activate Python virtual environment
cd ~/OAI/openairinterface5g/openair2/E2AP/flexric/build/examples/xApp/python3
python3 -m venv venv
source venv/bin/activate
```
Explanation: Create and activate an isolated Python environment to manage dependencies without affecting system Python.

```bash
# Upgrade pip and install Python libraries
pip install --upgrade pip
pip install pandas numpy
```
Explanation: Update Python package manager and install libraries for data manipulation and numerical computations.

```bash
# Run the Data Collector Script
python3 data_collector.py
```
Explanation: Executes the data collection script that gathers measurement and performance data from the running 5G testbed.
</details>
