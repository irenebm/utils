# FastDDS Inter-Container Discovery Issue

## Problem Summary
- ✅ FastDDS profile loads correctly (changes take effect)
- ✅ Intra-container communication works (talker/listener in same devkit container) 
- ❌ Inter-container communication fails with unicast (devkit ↔ robot/robot-viz)
- ✅ Inter-container communication works when multicast is enabled

## Root Cause: Initial Peers List
Your FastDDS config specifies these initial peers:
```xml
<initialPeersList>
    <locator>
        <udpv4>
            <address>192.168.56.192</address>
        </udpv4>
    </locator>
    <locator>
        <udpv4>
            <address>192.168.56.200</address>
        </udpv4>
    </locator>
</initialPeersList>
```

## The Issue
Since all containers use `network_mode: host`, they're all on the **host network**. The IP addresses `192.168.56.192` and `192.168.56.200` might be:

1. **Wrong network addresses** for your current setup
2. **Static addresses** that don't match your actual host IP
3. **Missing the devkit container's host IP** in the peers list

## Solutions

### Solution 1: Use Host IP Address
Find your actual host IP and update the config:
```bash
# Find your host IP
ip route get 1.1.1.1 | awk '{print $7; exit}'
# or
hostname -I | awk '{print $1}'
```

Then update your FastDDS config to use the actual host IP:
```xml
<initialPeersList>
    <locator>
        <udpv4>
            <address>YOUR_ACTUAL_HOST_IP</address>
        </udpv4>
    </locator>
</initialPeersList>
```

### Solution 2: Use Localhost/Any Address
Since all containers are on host network:
```xml
<initialPeersList>
    <locator>
        <udpv4>
            <address>127.0.0.1</address>
        </udpv4>
    </locator>
    <locator>
        <udpv4>
            <address>0.0.0.0</address>
        </udpv4>
    </locator>
</initialPeersList>
```

### Solution 3: Add Metatraffic Locator
Your config has an empty metatraffic locator. Try specifying it:
```xml
<metatrafficUnicastLocatorList>
    <locator>
        <udpv4>
            <address>0.0.0.0</address>
            <port>7400</port>
        </udpv4>
    </locator>
</metatrafficUnicastLocatorList>
```

### Solution 4: Dynamic Peers Discovery
Use environment variables to set peers dynamically:
```yaml
environment:
  - FASTRTPS_DEFAULT_PROFILES_FILE=/WorkingData/fastdds.xml
  - ROS_STATIC_PEERS=127.0.0.1
```

## Debugging Commands
Run these in your containers to verify network setup:

```bash
# Check actual IP addresses
ip addr show
hostname -I

# Check if containers can reach each other
ping 192.168.56.192
ping 192.168.56.200

# Test FastDDS discovery with logging
FASTRTPS_LOG_LEVEL=Log::Kind::Info ros2 run demo_nodes_cpp talker
# Look for "Found remote participant" messages
```

## Why Multicast Works
With multicast enabled, FastDDS automatically discovers participants through multicast announcements (224.0.0.251) without needing the initial peers list. When multicast is disabled, it relies entirely on the `initialPeersList` for discovery.

## Quick Test
Temporarily change your FastDDS config to use localhost:
```xml
<initialPeersList>
    <locator>
        <udpv4>
            <address>127.0.0.1</address>
        </udpv4>
    </locator>
</initialPeersList>
```

This should work since all containers are on the host network.