# SDN Path Tracing Tool - Setup Instructions for Fedora

This guide covers installing Mininet and Ryu on Fedora Linux from scratch.

## Prerequisites

- Fedora Linux (tested on Fedora 35/36/37/38)
- Root/sudo access
- Internet connection

---

## Step 1: Update System and Install Base Packages

```bash
# Update system package database
sudo dnf update -y

# Install required base packages
sudo dnf install -y python3 python3-pip git gcc make
```

---

## Step 2: Install Open vSwitch

Open vSwitch is required for SDN virtualization.

```bash
# Install Open vSwitch
sudo dnf install -y openvswitch

# Enable and start Open vSwitch service
sudo systemctl enable openvswitch.service
sudo systemctl start openvswitch.service

# Check service status
sudo systemctl status openvswitch.service
```

**Expected output:**
```
● openvswitch.service - Open vSwitch
     Loaded: loaded (/usr/lib/systemd/system/openvswitch.service; enabled)
     Active: active (exited)
```

---

## Step 3: Install Ryu Controller

Install Ryu controller using pip3.

```bash
# Install Ryu controller
sudo pip3 install ryu

# Verify installation
ryu --version
```

**Expected output:**
```
ryu 4.34
```

---

## Step 4: Install Mininet (From Source)

Mininet is not available in Fedora repos, so install from source.

```bash
# Clone Mininet repository
cd ~
git clone https://github.com/mininet/mininet.git
cd mininet

# Install Mininet with all options
sudo ./util/install.sh -a
```

This script will:
- Install Mininet core
- Install Open Flow references
- Install dependencies
- Set up command line tools

**Note:** The `-a` option installs ALL dependencies including:
- Open vSwitch
- Core dependencies
- DISSectors (Wireshark)

---

## Step 5: Verify Mininet Installation

```bash
# Check Mininet version
sudo mn --version

# Verify Mininet can start (quick test)
sudo mn --test pinghost
```

**Expected output:**
```
*** https://github.com/mininet/mininet/wiki/Frequently-Asked-Questions
*** No default OpenFlow controller found
*** Adding pseudo-term for cli
*** Creating network
*** Adding host h1
*** Adding host h2
*** Adding switch s1
*** Adding link h1-s1
*** Adding link s1-h2
*** Testing h1 ping h2
*** Results: 0% dropped (2/2 received)
```

---

## Step 6: Quick Test (Optional)

A complete end-to-end test to verify the setup:

```bash
# Start Ryu controller (in background or separate terminal)
# ryu-manager path_tracer.py

# Start Mininet with remote controller
sudo mn --topo linear,3 --controller remote --mac

# In Mininet CLI:
mininet> pingall
```

---

## Troubleshooting

### Issue: Mininet won't start

```bash
# Check if Open vSwitch is running
sudo systemctl status openvswitch

# Restart Open vSwitch
sudo systemctl restart openvswitch
```

### Issue: Permission denied

```bash
# Make sure to use sudo
sudo mn
```

### Issue: No controller found

Make sure Ryu is running before starting Mininet:
```bash
# Terminal 1:
ryu-manager path_tracer.py

# Terminal 2:
sudo mn --topo linear,3 --controller remote --mac
```

---

## Installed Components Summary

| Component | Installation Method | Version Check |
|-----------|---------------------|---------------|
| Python3 | dnf | `python3 --version` |
| pip3 | dnf | `pip3 --version` |
| Git | dnf | `git --version` |
| GCC | dnf | `gcc --version` |
| Open vSwitch | dnf | `ovs-vsctl --version` |
| Ryu | pip3 | `ryu --version` |
| Mininet | Source | `sudo mn --version` |

---

## Next Steps

After installation, proceed to:
1. Run Ryu controller: `ryu-manager path_tracer.py`
2. Start Mininet: `sudo mn --topo linear,3 --controller remote --mac`
3. Test with: `pingall` or `h1 ping h2`