# FastDDS Service Discovery Issue in Devkit

## Pattern Analysis

### ❌ **Config 1**: `0.0.0.0:7400` in initialPeersList
```xml
<initialPeersList>
    <locator>
        <udpv4>
            <address>0.0.0.0</address>
            <port>7400</port>
        </udpv4>
    </locator>
</initialPeersList>
```
**Result**: Services can't find each other
- `service worldstate/GetActiveAgent not available`
- `service camsmanager/SetCamsWorkingInfo not available`

### ✅ **Config 2**: `192.168.56.200` in initialPeersList (BEST)
```xml
<initialPeersList>
    <locator>
        <udpv4>
            <address>192.168.56.200</address>
        </udpv4>
    </locator>
</initialPeersList>
```
**Result**: "seems to work better" - services discover each other

### ❌ **Config 3**: `0.0.0.0:7400` in metatrafficUnicastLocatorList
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
**Result**: Services can't find each other again

## Root Cause

**Devkit container requires explicit self-reference in `initialPeersList` for intra-container service discovery.**

Unlike robot-viz containers, devkit needs:
- ✅ `<address>192.168.56.200</address>` (container's own IP)
- ❌ NOT `<address>0.0.0.0</address>`
- ❌ NOT empty `<locator/>`

## The Working Configuration

Based on your tests, this should work best for devkit:

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<profiles xmlns="http://www.eprosima.com/XMLSchemas/fastRTPS_Profiles">
    <participant profile_name="disable_multicast" is_default_profile="true">
        <rtps>
            <sendSocketBufferSize>12582912</sendSocketBufferSize>
            <listenSocketBufferSize>12582912</listenSocketBufferSize>
            <builtin>
                <metatrafficUnicastLocatorList>
                    <locator/>
                </metatrafficUnicastLocatorList>
                <initialPeersList>
                    <locator>
                        <udpv4>
                            <address>192.168.56.200</address>  <!-- devkit self-reference -->
                        </udpv4>
                    </locator>
                    <locator>
                        <udpv4>
                            <address>4.53.151.35</address>     <!-- for inter-container -->
                        </udpv4>
                    </locator>
                </initialPeersList>
            </builtin>
        </rtps>
    </participant>
</profiles>
```

## Why This Happens in Devkit

The devkit container has some difference (likely in FastDDS version, build configuration, or network stack) that requires explicit self-discovery. This could be due to:

1. **Different FastDDS build flags** in devkit dockerfile
2. **Different network interface binding** behavior
3. **Different default discovery mechanism**

## Remaining Issues

The `robot_description` error is **separate** from FastDDS:
```
Could not find parameter robot_description and did not receive robot_description via std_msgs::msg::String subscription
```

This indicates:
- Missing URDF parameter or publisher
- Unrelated to FastDDS configuration
- Likely a launch file or parameter configuration issue

## Solution

Use the configuration that works best (Config 2) with both self-reference and inter-container IPs:

```xml
<initialPeersList>
    <locator>
        <udpv4>
            <address>192.168.56.200</address>  <!-- REQUIRED for devkit intra-container -->
        </udpv4>
    </locator>
    <locator>
        <udpv4>
            <address>4.53.151.35</address>     <!-- for robot-viz communication -->
        </udpv4>
    </locator>
</initialPeersList>
```

This gives you both intra-container service discovery AND inter-container communication.