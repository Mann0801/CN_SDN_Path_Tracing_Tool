# SDN Path Tracing Tool

A complete SDN-based Path Tracing Tool using Mininet and Ryu Controller on Fedora Linux.

---

## Project Overview

This project demonstrates:
- **SDN Controller-Switch Interaction** using OpenFlow 1.3
- **Path Tracing** - tracking packets across multiple switches
- **Flow Rule Installation** using OFPFlowMod
- **Mininet Emulation** with linear topology

---

## Files Included

| File | Description |
|------|-------------|
| `path_tracer.py` | Ryu controller code |
| `SETUP.md` | Fedora installation guide |
| `MININET.md` | Topology instructions |
| `FLOW_RULES.md` | Flow rule implementation details |
| `EXECUTION.md` | Step-by-step execution guide |
| `TESTING.md` | Testing & validation guide |
| `README.md` | This file |

---

## Quick Start

### 1. Install Dependencies (Fedora)

```bash
# Install packages
sudo dnf update -y
sudo dnf install -y python3 python3-pip git gcc make openvswitch
sudo pip3 install ryu

# Clone and install Mininet
cd ~
git clone https://github.com/mininet/mininet.git
cd mininet
sudo ./util/install.sh -a
```

### 2. Run Controller

```bash
# Terminal 1
ryu-manager path_tracer.py
```

### 3. Run Mininet

```bash
# Terminal 2
sudo mn --topo linear,3 --controller remote --mac
```

### 4. Test

```bash
mininet> pingall
mininet> h1 ping h2
mininet> dpctl dump-flows
```

---

## Expected Output

When h1 pings h4 (through 3 switches):

```
[INFO] Packet: 10.0.0.1 -> 10.0.0.4 | Switch: s1
[INFO] Path from 10.0.0.1 to 10.0.0.4: s1 -> s2 -> s3
```

---

## Topology

```
h1(10.0.0.1) --- s1 --- s2 --- s3 --- h2(10.0.0.2)
                     |        |
                    h3(10.0.0.3)  h4(10.0.0.4)
```

## License

Educational project for Computer Networks (CN) course.
