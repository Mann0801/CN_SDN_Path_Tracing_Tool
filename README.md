# SDN Path Tracing Tool (Ryu + Mininet)

## Project Overview

This project implements a Software Defined Networking (SDN) application using the Ryu controller and Mininet to demonstrate:

* Controller–switch interaction using OpenFlow 1.3
* Handling of Packet-In events
* Flow rule installation using OFPFlowMod
* Path tracing of packets across multiple switches
* Network behavior under normal and failure conditions

---

## Components Used

* Mininet – Network emulator (hosts, switches, links)
* Ryu – SDN controller (control plane)
* OpenFlow – Protocol for communication between controller and switches
* Python 3.11 – Required for Ryu compatibility

---

## Topology

Linear topology with three switches:

```
h1 (10.0.0.1) --- s1 --- s2 --- s3 --- h2 (10.0.0.2)
```

* s1, s2, s3 represent OpenFlow-enabled switches
* h1 and h2 represent end hosts

---

## Setup Instructions (Fedora)

### 1. Install Python 3.11

```
sudo dnf install -y python3.11
```

### 2. Install Ryu and dependencies

```
python3.11 -m ensurepip --upgrade
python3.11 -m pip install --upgrade pip setuptools wheel
python3.11 -m pip install --user ryu netaddr eventlet msgpack oslo.config ovs routes six tinyrpc webob "packaging<21"
```

### 3. Install Mininet

```
cd ~
git clone https://github.com/mininet/mininet.git
cd mininet
sudo ./util/install.sh -fnpv
```

---

## Execution Steps

### Step 1: Start Controller (Terminal 1)

```
cd ~/Documents/SEM_4_STUFF/CN/sdn
python3.11 -m ryu.cmd.manager path_tracer.py
```

### Step 2: Start Mininet (Terminal 2)

```
sudo mn -c
sudo mn --topo linear,3 --controller remote
```

### Step 3: Generate Traffic

Inside Mininet:

```
h1 ping -c 5 h2
```

---

## Expected Output

### Mininet

```
0% packet loss
```

### Controller (Ryu)

```
Path 10.0.0.1 -> 10.0.0.2: s1 -> s2 -> s3
```

---

## Working Principle

1. A host sends a packet to the network
2. The switch, lacking a matching flow entry, sends a Packet-In message to the controller
3. The controller processes the packet and learns MAC address mappings
4. The controller installs a flow rule using Flow-Mod
5. The packet is forwarded through the switches
6. The path taken by the packet is recorded and displayed

---

## Testing and Validation

### Scenario 1: Normal Communication

* Command: `h1 ping h2`
* Result: Successful communication
* Output: Path traced as s1 -> s2 -> s3

---

### Scenario 2: Link Failure

Inside Mininet:

```
link s1 s2 down
```

Then:

```
h1 ping h2
```

Result:

```
Destination Host Unreachable
```

Explanation:
The linear topology does not provide an alternative path, resulting in communication failure.

---

## Key Concepts Demonstrated

* Separation of control plane and data plane
* Centralized network control using an SDN controller
* OpenFlow-based communication
* Flow table rule installation
* Dynamic MAC address learning
* Packet path tracking across switches

---

## Observations

* Initial packets are flooded before flow rules are installed
* Subsequent packets follow established flow rules
* Path output may include repeated switches due to continuous packet processing

---

## Conclusion

This project demonstrates the core principles of Software Defined Networking, including centralized control, dynamic flow rule management, and visibility into packet paths. It also highlights the impact of topology constraints on network reliability.

---

## Files Included

* `path_tracer.py` – Ryu controller implementation
* `README.md` – Project documentation

---

## Author

SDN Project – Computer Networks Course
