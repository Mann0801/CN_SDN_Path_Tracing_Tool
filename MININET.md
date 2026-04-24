# Mininet Topology Instructions

## Linear Topology with 3 Switches

The topology used in this project:

```
        10.0.0.1          10.0.0.2          10.0.0.3          10.0.0.4
           |                |                |                |
           h1               h2               h3               h4
           |                |                |                |
        s1(0001) -------- s2(0002) -------- s3(0003) -------- s4(0004)

Linear topology: h1-s1, s1-s2, s2-s3, s3-h4
```

---

## Mininet Command for This Project

### Standard Linear Topology (3 switches, 4 hosts)

```bash
sudo mn --topo linear,3 --controller remote --mac
```

**Explanation:**
- `--topo linear,3`: Creates a linear topology with 3 switches
- `--controller remote`: Connects to a remote controller (Ryu)
- `--mac`: Sets MAC addresses to be deterministic (easier to read)

---

## Alternative Topologies

### Single Switch with 2 Hosts

```bash
sudo mn --topo single,2 --controller remote --mac
```

### Tree Topology

```bash
sudo mn --topo tree,depth=2 --controller remote --mac
```

### Custom Topology (Python)

Create a custom topology file:

```python
# custom_topo.py
from mininet.topo import Topo

class MyTopo(Topo):
    def __init__(self):
        Topo.__init__(self)

        # Add switches
        s1 = self.addSwitch('s1')
        s2 = self.addSwitch('s2')
        s3 = self.addSwitch('s3')

        # Add hosts
        h1 = self.addHost('h1', ip='10.0.0.1/24')
        h2 = self.addHost('h2', ip='10.0.0.2/24')

        # Add links
        self.addLink(h1, s1)
        self.addLink(s1, s2)
        self.addLink(s2, s3)
        self.addLink(s3, h2)

topos = {'mytopo': MyTopo}
```

Run with:
```bash
sudo python3 custom_topo.py
```

---

## Mininet CLI Commands

Once Mininet is running, use these commands:

### Basic Commands

```bash
# List all nodes
mininet> nodes

# Show network links
mininet> net

# Show IP addresses
mininet> dump

# Test connectivity between all hosts
mininet> pingall

# Ping between specific hosts
mininet> h1 ping h2
```

### Testing Commands

```bash
# Test bandwidth using iperf
mininet> iperf

# Test bandwidth between specific hosts
mininet> h1 iperf h2

# View flow tables on all switches
mininet> dpctl dump-flows

# View flow tables on specific switch
mininet> s1 dpctl dump-flows
```

### Debugging Commands

```summary
# Show detailed node info
mininet> h1 ifconfig

# Run command on host
mininet> h1 ip route

# Start wireshark on switch
mininet> s1 wireshark &
```

---

## Expected Host IP Addresses

In our linear,3 topology:

| Host | IP Address   | MAC Address     | Connected To |
|------|--------------|-----------------|--------------|
| h1   | 10.0.0.1     | 00:00:00:00:00:01 | s1         |
| h2   | 10.0.0.2     | 00:00:00:00:00:02 | s1         |
| h3   | 10.0.0.3     | 00:00:00:00:00:03 | s2         |
| h4   | 10.0.0.4     | 00:00:00:00:00:04 | s3         |

Wait, that's not right. Let me recalculate for linear,3:

```
linear,3 = 3 switches, 4 hosts
h1 -- s1 -- s2 -- s3 -- h2

h1: 10.0.0.1 (connected to s1)
h2: 10.0.0.2 (connected to s1)
h3: 10.0.0.3 (connected to s2)
h4: 10.0.0.4 (connected to s3)
```

With `--mac` flag:
- h1: 00:00:00:00:00:01
- h2: 00:00:00:00:00:02
- h3: 00:00:00:00:00:03
- h4: 00:00:00:00:00:04

---

## Exit Mininet

```bash
# Exit Mininet CLI
mininet> exit

# Or kill all processes
sudo mn -c
```