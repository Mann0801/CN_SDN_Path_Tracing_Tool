# Execution Steps

This guide provides step-by-step instructions to run the SDN Path Tracing Tool.

---

## Prerequisites

Before starting, ensure you have:

1. ✅ Mininet installed
2. ✅ Ryu controller installed
3. ✅ Open vSwitch running
4. ✅ `path_tracer.py` controller code ready

---

## Step 1: Start Open vSwitch

First, ensure Open vSwitch is running:

```bash
# Check if Open vSwitch is running
sudo systemctl status openvswitch

# If not running, start it
sudo systemctl start openvswitch
```

---

## Step 2: Run Ryu Controller

Open a terminal and start the Ryu controller with path_tracer.py:

```bash
# Navigate to the project directory (if not already there)
cd /home/mannmehta/Documents/SEM_4_STUFF/CN/sdn

# Run Ryu controller
ryu-manager path_tracer.py
```

**Expected Output:**
```
loading app path_tracer.py
loading app ryu.controller.ofp_event
loading app ryu.controller.handler
loading app ryu.lib.packet
loading app ryu.lib.ofctl_v1_3
instantiating app ryu.controller.ofp_event
instantiating app ryu.controller.handler
instantiating app ryu.lib.packet
instantiating app ryu.controller.dpset
instantiating app ryu.lib.ofctl_v1_3
instantiating app path_tracer.py
```

The controller is now listening on port 6653 (OpenFlow default).

**Keep this terminal open!**

---

## Step 3: Start Mininet

Open a **new terminal** and create the linear topology:

```bash
# Start Mininet with linear topology
sudo mn --topo linear,3 --controller remote --mac
```

**Expected Output:**
```
*** Unable to contact the remote controller (127.0.0.1:6633)
*** Remote controller (ryu) failed
*** Adding pseudo-term for cli
*** Creating network
*** Adding host h1
*** Adding host h2
*** Adding host h3
*** Adding host h4
*** Adding switch s1
*** Adding switch s2
*** Adding switch s3
*** Adding link h1-s1
*** Adding link h1-h2
*** Adding link s1-s2
*** Adding link s2-s3
*** Adding link s3-h4
*** Starting Controller
*** Starting CLI:
mininet>
```

The controller connection might initially fail, but Ryu should connect automatically.

---

## Step 4: Verify Controller Connection

In the Mininet CLI, verify that switches are connected:

```bash
mininet> nodes
```

**Expected:**
```
available nodes are:
h1 h2 h3 h4 s1 s2 s3
```

Check connections:
```bash
mininet> net
```

---

## Step 5: Run Tests

### Test 1: Ping All

```bash
mininet> pingall
```

This will send ICMP packets between all hosts. The controller will track the paths.

**Expected Output in Ryu Terminal:**
```
[INFO] Packet: 10.0.0.1 -> 10.0.0.2 | Switch: s1
[INFO] Path from 10.0.0.1 to 10.0.0.2: s1
[INFO] Packet: 10.0.0.2 -> 10.0.0.1 | Switch: s1
[INFO] Path from 10.0.0.2 to 10.0.0.1: s1
[INFO] Packet: 10.0.0.1 -> 10.0.0.3 | Switch: s1
[INFO] Path from 10.0.0.1 to 10.0.0.3: s1 -> s2
[INFO] Packet: 10.0.0.3 -> 10.0.0.1 | Switch: s2
[INFO] Path from 10.0.0.3 to 10.0.0.1: s2 -> s1
[INFO] Packet: 10.0.0.1 -> 10.0.0.4 | Switch: s1
[INFO] Path from 10.0.0.1 to 10.0.0.4: s1 -> s2 -> s3
[INFO] Packet: 10.0.0.4 -> 10.0.0.1 | Switch: s3
[INFO] Path from 10.0.0.4 to 10.0.0.1: s3 -> s2 -> s1
```

### Test 2: Ping Between Specific Hosts

```bash
mininet> h1 ping -c 3 h2
```

**Expected:**
```
PING 10.0.0.2 (10.0.0.2) 56(84) bytes of data.
64 bytes from 10.0.0.2: icmp_seq=1 ttl=64 time=... ms
64 bytes from 10.0.0.2: icmp_seq=2 ttl=64 time=... ms
64 bytes from 10.0.0.2: icmp_seq=3 ttl=64 time=... ms

--- 10.0.0.2 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2000ms
```

**Path Output in Ryu Terminal:**
```
[INFO] Packet: 10.0.0.1 -> 10.0.0.2 | Switch: s1
[INFO] Path from 10.0.0.1 to 10.0.0.2: s1
```

### Test 3: Ping Across Multiple Switches

```bash
mininet> h1 ping -c 3 h4
```

**Expected Output in Ryu Terminal:**
```
[INFO] Packet: 10.0.0.1 -> 10.0.0.4 | Switch: s1
[INFO] Path from 10.0.0.1 to 10.0.0.4: s1 -> s2 -> s3
```

---

## Step 6: View Flow Tables

### View All Flow Rules

```bash
mininet> dpctl dump-flows
```

**Expected Output:**
```
s1
 cookie=0x0, duration=...s, table=0, n_packets=..., n_bytes=..., priority=100,ip,nw_dst=10.0.0.2 actions=output:2,flood
 cookie=0x0, duration=...s, table=0, n_packets=..., n_bytes=..., priority=100,ip,nw_dst=10.0.0.3 actions=output:3,flood
 cookie=0x0, duration=...s, table=0, n_packets=..., n_bytes=..., priority=100,ip,nw_dst=10.0.0.4 actions=output:3,flood
s2
 cookie=0.0, duration=...s, table=0, n_packets=..., n_bytes=..., priority=100,ip,nw_dst=10.0.0.1 actions=output:1,flood
 cookie=0.0, duration=...s, table=0, n_packets=..., n_bytes=..., priority=100,ip,nw_dst=10.0.0.4 actions=output:3,flood
s3
 ...
```

### View Specific Switch

```bash
mininet> s1 dpctl dump-flows
```

---

## Step 7: Run iperf Test

```bash
mininet> iperf
```

This runs a simple TCP bandwidth test between h1 and h2.

**Expected:**
```
*** Iperf: testing TCP bandwidth between h1 and h2
*** Results: ['10.0 Mbits/sec', '9.8 Mbits/sec']
```

---

## Step 8: Cleanup

When finished, exit Mininet properly:

```bash
mininet> exit
```

Or clean up all processes:

```bash
sudo mn -c
```

---

## Summary of Commands

| Action | Command |
|--------|---------|
| Start Controller | `ryu-manager path_tracer.py` |
| Start Mininet | `sudo mn --topo linear,3 --controller remote --mac` |
| Ping All | `pingall` |
| Ping Host to Host | `h1 ping h2` |
| View Flow Tables | `dpctl dump-flows` |
| Run iperf | `iperf` |
| Show Nodes | `nodes` |
| Show Network | `net` |
| Exit Mininet | `exit` |