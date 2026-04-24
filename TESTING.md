# Testing & Validation Guide

This document provides expected outputs and validation steps for the SDN Path Tracing Tool.

---

## Expected Output: Path Tracing

### Scenario 1: h1 to h2 (Same Switch)

```
mininet> h1 ping -c 1 h2
PING 10.0.0.2 (10.0.0.2) 56(84) bytes of data.
64 bytes from 10.0.0.2: icmp_seq=1 ttl=64 time=... ms

--- 10.0.0.2 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
```

**Ryu Controller Output:**
```
[INFO] Packet: 10.0.0.1 -> 10.0.0.2 | Switch: s1
[INFO] Path from 10.0.0.1 to 10.0.0.2: s1
```

### Scenario 2: h1 to h3 (Two Switches)

```
mininet> h1 ping -c 1 h3
```

**Ryu Controller Output:**
```
[INFO] Packet: 10.0.0.1 -> 10.0.0.3 | Switch: s1
[INFO] Path from 10.0.0.1 to 10.0.0.3: s1 -> s2
```

### Scenario 3: h1 to h4 (Three Switches)

```
mininet> h1 ping -c 1 h4
```

**Ryu Controller Output:**
```
[INFO] Packet: 10.0.0.1 -> 10.0.0.4 | Switch: s1
[INFO] Path from 10.0.0.1 to 10.0.0.4: s1 -> s2 -> s3
```

---

## Expected Output: Flow Tables

### View Flow Rules (dpctl)

```bash
mininet> dpctl dump-flows
```

**Expected Output:**
```
s1
 cookie=0x0, duration=12.345s, table=0, n_packets=3, n_bytes=294, priority=100,ip,nw_dst=10.0.0.2 actions=output:2,flood
 cookie=0x0, duration=10.234s, table=0, n_packets=3, n_bytes=294, priority=100,ip,nw_dst=10.0.0.3 actions=output:3,flood
 cookie=0x0, duration=8.123s, table=0, n_packets=3, n_bytes=294, priority=100,ip,nw_dst=10.0.0.4 actions=output:3,flood

s2
 cookie=0.0, duration=11.234s, table=0, n_packets=3, n_bytes=294, priority=100,ip,nw_dst=10.0.0.1 actions=output:1,flood
 cookie=0.0, duration=9.876s, table=0, n_packets=3, n_bytes=294, priority=100,ip,nw_dst=10.0.0.4 actions=output:3,flood

s3
 cookie=0.0, duration=10.543s, table=0, n_packets=3, n_bytes=294, priority=100,ip,nw_dst=10.0.0.1 actions=output:1,flood
```

**Field Explanations:**
- `cookie`: Opaque identifier (0x0 = default)
- `duration`: How long the flow has been active
- `n_packets`: Number of packets matched
- `n_bytes`: Total bytes matched
- `priority`: Match priority (higher = more specific)
- `ip,nw_dst=...`: Match criteria (destination IP)
- `actions=...`: What to do with matching packets

---

## Expected Output: pingall

```bash
mininet> pingall
```

**Expected Output:**
```
*** Ping: testing reachability over IPv4
*** 
h1 -> h2 h3 h4 
h2 -> h1 h3 h4 
h3 -> h1 h2 h4 
h4 -> h1 h2 h3 
*** Results: 0% dropped (16/16 received)
```

**Ryu Controller Output:**
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
[INFO] Packet: 10.0.0.2 -> 10.0.0.3 | Switch: s1
[INFO] Path from 10.0.0.2 to 10.0.0.3: s1 -> s2
[INFO] Packet: 10.0.0.3 -> 10.0.0.2 | Switch: s2
[INFO] Path from 10.0.0.3 to 10.0.0.2: s2 -> s1
[INFO] Packet: 10.0.0.2 -> 10.0.0.4 | Switch: s1
[INFO] Path from 10.0.0.2 to 10.0.0.4: s1 -> s2 -> s3
[INFO] Packet: 10.0.0.4 -> 10.0.0.2 | Switch: s3
[INFO] Path from 10.0.0.4 to 10.0.0.2: s3 -> s2 -> s1
[INFO] Packet: 10.0.0.3 -> 10.0.0.4 | Switch: s2
[INFO] Path from 10.0.0.3 to 10.0.0.4: s2 -> s3
[INFO] Packet: 10.0.0.4 -> 10.0.0.3 | Switch: s3
[INFO] Path from 10.0.0.4 to 10.0.0.3: s3
```

---

## Expected Output: iperf

```bash
mininet> iperf
```

**Expected Output:**
```
*** Iperf: testing TCP bandwidth between h1 and h2
*** Results: ['10.0 Mbits/sec', '9.8 Mbits/sec']
*** 
*** Iperf: testing TCP bandwidth between h3 and h4
*** Results: ['10.0 Mbits/sec', '9.8 Mbits/sec']
```

---

## Validation Checklist

### Pre-Run Checklist
- [ ] Open vSwitch is running (`sudo systemctl status openvswitch`)
- [ ] Ryu controller starts without errors
- [ ] Mininet topology creates successfully

### Run-Time Validation
- [ ] Ryu shows "loading app path_tracer.py"
- [ ] Switches connect to controller (check Ryu output)
- [ ] Ping between hosts works (0% packet loss)
- [ ] Path tracing shows correct switch sequence

### Post-Run Validation
- [ ] Flow rules installed in switches (`dpctl dump-flows`)
- [ ] Flow counters increment (n_packets > 0)
- [ ] No errors in Ryu controller output

---

## Troubleshooting

### Issue: No Path Output

**Problem:** Ryu controller doesn't print path information.

**Solution:**
1. Check if controller is connected to switches
2. Verify packets are being sent (ping test)
3. Check logging level in code (should be INFO)

### Issue: Flow Rules Not Installed

**Problem:** `dpctl dump-flows` shows no flows.

**Solution:**
1. Generate traffic (ping, iperf)
2. Check Ryu output for flow installation messages
3. Verify controller can communicate with switches

### Issue: 100% Packet Loss

**Problem:** Ping fails between hosts.

**Solution:**
1. Check if controller is running
2. Verify network topology is correct
3. Run `dpctl dump-flows` to see installed rules

### Issue: Controller Connection Failed

**Problem:** Mininet can't connect to Ryu controller.

**Solution:**
1. Ensure Ryu is running first
2. Check Ryu is listening on port 6653
3. Try `sudo mn --controller remote,ip=127.0.0.1,port=6653`

---

## Additional Testing Commands

### View Switch Details

```bash
# View switch info
mininet> s1 ifconfig

# View host details
mininet> h1 ifconfig

# View all links
mininet> net

# View all nodes
mininet> nodes
```

### Continuous Ping

```bash
mininet> h1 ping -t h4
```

This will continuously ping h4 from h1, showing path updates.

### OpenFlow Protocol Messages

Use Wireshark to capture OpenFlow messages:

```bash
# Install Wireshark
sudo dnf install -y wireshark

# Capture on loopback
sudo wireshark -i lo -k &
```

Filter by `of` or `openflow` protocol.

---

## Clean Up

Always clean up after testing:

```bash
# Exit Mininet
mininet> exit

# Clean all Mininet processes
sudo mn -c
```

---

## Summary

| Test | Expected Result |
|------|-----------------|
| `pingall` | 0% packet loss |
| `h1 ping h4` | Path: s1 -> s2 -> s3 |
| `dpctl dump-flows` | Flow rules visible |
| `iperf` | Bandwidth ~10 Mbits/sec |