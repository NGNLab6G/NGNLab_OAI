# OAI FlexRIC 5G Testbed Usage Guide

```bash
# Start OAI 5G Core
cd ~/OAI/oai-cn5g
docker compose up -d
```
```bash
docker compose ps -a
```
Explanation: Navigates to the OAI Core directory and starts all 5G core containers in the background. Displays the status of all containers to ensure they are running successfully.

```bash
# In new terminal
# Start the near-RT RIC
cd ~/OAI/openairinterface5g/openair2/E2AP/flexric/build/examples/ric
./nearRT-RIC
# Keep this terminal open
```
Explanation: Navigates to the FlexRIC build directory and launches the near-real-time RIC, enabling E2 connections and xApp control.

```bash
# In new terminal
# Start OAI gNB with E2-Agent
cd ~/OAI/openairinterface5g/cmake_targets/ran_build/build
sudo ./nr-softmodem -O ~/OAI/openairinterface5g/targets/PROJECTS/GENERIC-NR-5GC/CONF/gnb.sa.band78.fr1.106PRB.usrpb210.conf \
  --rfsim \
  --e2_agent.near_ric_ip_addr 127.0.0.1 \
  --e2_agent.sm_dir /usr/local/lib/flexric/ \
  --gNBs.[0].min_rxtxtime 6
# Keep this terminal open
```
Explanation: Starts the gNB and connects it to the 5G Core and the nearRT-RIC. Enables E2-Agent to receive control and monitoring commands from FlexRIC.

```bash
# In new terminal
# Start UE1
sudo ~/OAI/openairinterface5g/tools/scripts/multi-ue.sh -c1
sudo ~/OAI/openairinterface5g/tools/scripts/multi-ue.sh -o1
```
```bash
sudo ./nr-uesoftmodem \
  -r 106 --numerology 1 --band 78 -C 3619200000 \
  --rfsim \
  --uicc0.imsi 001010000000001 \
  --rfsimulator.serveraddr 10.201.1.100
# Keep this terminal open
```
Explanation: Creates namespace for UE1, starts logging, and launches first NR UE emulator with unique IMSI and virtual RF interface.

Use similar commands updating the IMSI and RF simulator IP accordingly for each UE.

```bash
# In new terminal
# Start UE2
sudo ~/OAI/openairinterface5g/tools/scripts/multi-ue.sh -c2
sudo ~/OAI/openairinterface5g/tools/scripts/multi-ue.sh -o2
```
```bash
sudo ./nr-uesoftmodem \
  -r 106 --numerology 1 --band 78 -C 3619200000 \
  --rfsim \
  --uicc0.imsi 001010000000002 \
  --rfsimulator.serveraddr 10.202.1.100
# Keep this terminal open
```

```bash
# In new terminal
# Start UE3
sudo ~/OAI/openairinterface5g/tools/scripts/multi-ue.sh -c3
sudo ~/OAI/openairinterface5g/tools/scripts/multi-ue.sh -o3
```
```bash
sudo ./nr-uesoftmodem \
  -r 106 --numerology 1 --band 78 -C 3619200000 \
  --rfsim \
  --uicc0.imsi 001010000000003 \
  --rfsimulator.serveraddr 10.203.1.100
# Keep this terminal open
```

```bash
# In new terminal
# Launch xApps using C
cd ~/OAI/openairinterface5g/openair2/E2AP/flexric/build/examples/xApp/c
```
Explanation: Switch to the directory where C-based xApps are located.

```bash
# Run C xApp
XAPP_DURATION=30 ./xapp_name
```
Explanation: Runs a compiled C-based xApp executable for 30 seconds.

XAPP_DURATION environment variable overwrites the default xApp duration of 20 seconds. If the negative value is used, the xApp duration is considered to be infinite.

```bash
# In new terminal
# Launch xApps using Python
cd ~/OAI/openairinterface5g/openair2/E2AP/flexric/build/examples/xApp/python3
```
Explanation: Switch to the directory where Python-based xApps are located.

```bash
# Run Python xApp
XAPP_DURATION=30 python3 xapp_name
```
Explanation: Runs a Python-based xApp for 30 seconds.

```bash
# Python Virtual Environment
python3 -m venv venv
source venv/bin/activate
```
Explanation: Creates and activates an isolated Python environment to avoid dependency conflicts when running xApps.

```bash
# Run AI/ML xApp
python3 xapp_name
```
Explanation: Runs a Python-based AI/ML xApp.
