# Flow Rule Implementation Details

This document explains how flow rules are implemented in the Path Tracer controller.

---

## OpenFlow 1.3 Flow Table

Flow rules in OpenFlow consist of:
1. **Match Fields**: Criteria for matching packets
2. **Priority**: Higher priority rules are matched first
3. **Instructions/Actions**: What to do with matching packets
4. **Counters**: Track how many packets/bytes matched
5. **Timeouts**: When the flow should expire

---

## Flow Rule Structure in path_tracer.py

### Creating Match Fields

```python
# For IPv4 packets
match = parser.OFPMatch(
    eth_type=0x0800,    # Ethernet type for IPv4
    ipv4_dst=dst_ip     # Destination IP address
)

# For ARP packets
match = parser.OFPMatch(
    eth_type=0x0806,    # Ethernet type for ARP
    eth_dst=dst_mac     # Destination MAC address
)
```

### Creating Actions

```python
# Flood action (send to all ports except input)
actions = [
    parser.OFPActionOutput(ofproto.OFPP_FLOOD)
]

# Specific port action
actions = [
    parser.OFPActionOutput(port_number)
]
```

### Creating Flow Mod Message

```python
# Build the flow modification message
mod = parser.OFPFlowMod(
    datapath=datapath,           # Target switch
    priority=100,                # Match priority (higher = more important)
    match=match,                 # Match criteria
    actions=actions,             # Actions to apply
    idle_timeout=60,             # Remove after 60s of no packets
    hard_timeout=300,           # Remove after 300s total
    instructions=[
        parser.OFPInstructionActions(
            ofproto.OFPIT_APPLY_ACTIONS,
            actions
        )
    ]
)

# Send to switch
datapath.send_msg(mod)
```

---

## Flow Rule Installation Process

### Step 1: Packet Arrives at Switch

When a packet arrives at a switch:

1. Switch checks its flow table
2. If no matching flow found, sends packet_in to controller

### Step 2: Controller Processes Packet

1. Controller receives packet_in event
2. Parses packet to extract headers (IP, MAC, etc.)
3. Determines appropriate action (forward, flood, drop)
4. Creates flow rule to handle similar packets

### Step 3: Install Flow Rule

1. Controller sends OFPFlowMod to switch
2. Switch installs flow rule in its table
3. Subsequent packets match the rule automatically
4. No more packet_in messages needed

### Step 4: Packet Forwarded

1. Next packet arrives at switch
2. Matches the installed flow rule
3. Action is applied immediately
4. Packet forwarded to destination

---

## Flow Rule Parameters Explained

| Parameter | Description | Our Value |
|-----------|-------------|-----------|
| `priority` | Match priority (0-65535) | 100 |
| `idle_timeout` | Seconds of inactivity before removal | 60 |
| `hard_timeout` | Seconds before forced removal | 300 |
| `cookie` | Opaque identifier for flow | 0 (default) |

---

## Default Forwarding: FLOOD

In this implementation, we use FLOOD as the default forwarding action:

```python
actions = [
    parser.OFPActionOutput(ofproto.OFPP_FLOOD)
]
```

**What FLOOD does:**
- Sends packet to all ports except the input port
- Used when destination is unknown (ARP, unknown MAC)
- Ensures packet reaches destination in unknown topology

**Why FLOOD:**
- Simple to implement
- Works without topology knowledge
- Guaranteed delivery in small networks

---

## Alternative Forwarding Strategies

### 1. MAC Learning (Not implemented)

Track which port each MAC address is connected to:

```python
self.mac_to_port[mac_address] = output_port

# Then forward to specific port
actions = [
    parser.OFPActionOutput(self.mac_to_port[dst_mac])
]
```

### 2. IP-based Forwarding (Not implemented)

Match on destination IP and forward to next hop:

```python
match = parser.OFPMatch(
    eth_type=0x0800,
    ipv4_dst=dst_ip
)

actions = [
    parser.OFPActionOutput(next_hop_port)
]
```

### 3. Shortest Path Forwarding (Advanced)

Calculate shortest path using Dijkstra's algorithm:

```python
path = self._calculate_shortest_path(src, dst)
actions = [
    parser.OFPActionOutput(path[0])  # First hop
]
```

---

## Viewing Installed Flow Rules

### In Mininet CLI

```bash
# View all flow tables
mininet> dpctl dump-flows

# View specific switch
mininet> s1 dpctl dump-flows

# Example output:
# s1
#  cookie=0x0, duration=10.5s, table=0, n_packets=5, n_bytes=490, priority=100,ip,nw_dst=10.0.0.2 actions=output:2,flood
```

### Using ovs-ofctl Command

```bash
# Direct command to switch
sudo ovs-ofctl dump-flows s1
```

---

## Flow Rule Lifecycle

```
      Packet arrives
           |
           v
    +--------------+
    | Check flow   |---- Yes ----> Apply action
    | table match? |
    +--------------+
           | No
           v
    +--------------+
    | Send to      |
    | controller   |
    +--------------+
           |
           v
    +--------------+
    | Install flow |
    | rule         |
    +--------------+
           |
           v
    Packet forwarded
           |
           v
    Flow rule active
           |
    +--------------+
    | Timeout?     |---- Yes ----> Remove flow
    +--------------+
```

---

## Flow Rule Expiration

Flow rules can expire in two ways:

1. **Idle Timeout** (60 seconds)
   - Removed after 60 seconds of no matching packets
   - Good for handling dynamic topologies

2. **Hard Timeout** (300 seconds)
   - Removed after 300 seconds regardless of traffic
   - Ensures old rules don't persist forever

In our implementation:
- Flow rules expire after 60 seconds of inactivity
- Maximum lifetime is 300 seconds