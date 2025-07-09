# Correct FastDDS Hybrid Configuration

## The Issue
My previous XML syntax was incorrect. Here's the **correct FastDDS configuration** for your needs:

## Solution 1: Multicast Discovery + Unicast Data (Recommended)

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<profiles xmlns="http://www.eprosima.com/XMLSchemas/fastRTPS_Profiles">
    <participant profile_name="hybrid_discovery" is_default_profile="true">
        <rtps>
            <sendSocketBufferSize>12582912</sendSocketBufferSize>
            <listenSocketBufferSize>12582912</listenSocketBufferSize>
            <builtin>
                <!-- Keep multicast for discovery -->
                <metatrafficMulticastLocatorList>
                    <locator>
                        <udpv4>
                            <address>239.255.0.1</address>
                            <port>7400</port>
                        </udpv4>
                    </locator>
                </metatrafficMulticastLocatorList>
                <!-- Add unicast peers for redundancy -->
                <initialPeersList>
                    <locator>
                        <udpv4>
                            <address>192.168.56.200</address>
                        </udpv4>
                    </locator>
                    <locator>
                        <udpv4>
                            <address>4.53.151.35</address>
                        </udpv4>
                    </locator>
                </initialPeersList>
            </builtin>
            <!-- Force user data to unicast only -->
            <userTransports>
                <transport_id>udp_transport</transport_id>
            </userTransports>
            <useBuiltinTransports>false</useBuiltinTransports>
        </rtps>
    </participant>
    
    <!-- Define custom UDP transport -->
    <transport_descriptors>
        <transport_descriptor>
            <transport_id>udp_transport</transport_id>
            <type>UDPv4</type>
            <maxMessageSize>65500</maxMessageSize>
            <maxInitialPeersRange>4</maxInitialPeersRange>
        </transport_descriptor>
    </transport_descriptors>
</profiles>
```

## Solution 2: Minimal Multicast (Simpler)

If the above is too complex, try this simpler approach:

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<profiles xmlns="http://www.eprosima.com/XMLSchemas/fastRTPS_Profiles">
    <participant profile_name="minimal_multicast" is_default_profile="true">
        <rtps>
            <sendSocketBufferSize>12582912</sendSocketBufferSize>
            <listenSocketBufferSize>12582912</listenSocketBufferSize>
            <builtin>
                <!-- Enable multicast on a restricted address -->
                <metatrafficMulticastLocatorList>
                    <locator>
                        <udpv4>
                            <address>239.255.0.1</address>
                            <port>7400</port>
                        </udpv4>
                    </locator>
                </metatrafficMulticastLocatorList>
            </builtin>
        </rtps>
    </participant>
</profiles>
```

## Solution 3: Network-Specific Multicast

Use your actual network for multicast:

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<profiles xmlns="http://www.eprosima.com/XMLSchemas/fastRTPS_Profiles">
    <participant profile_name="network_multicast" is_default_profile="true">
        <rtps>
            <sendSocketBufferSize>12582912</sendSocketBufferSize>
            <listenSocketBufferSize>12582912</listenSocketBufferSize>
            <builtin>
                <metatrafficMulticastLocatorList>
                    <locator>
                        <udpv4>
                            <address>224.0.0.251</address>
                            <port>7400</port>
                        </udpv4>
                    </locator>
                </metatrafficMulticastLocatorList>
                <metatrafficUnicastLocatorList>
                    <locator>
                        <udpv4>
                            <address>192.168.56.200</address>
                            <port>7401</port>
                        </udpv4>
                    </locator>
                </metatrafficUnicastLocatorList>
            </builtin>
        </rtps>
    </participant>
</profiles>
```

## Key Insight

Your testing showed that:
- **Empty `<builtin></builtin>`** = Full multicast = ✅ Everything works
- **Unicast-only with peers** = ❌ Service discovery breaks

The solution is to **keep multicast for discovery** but you can still **optimize data paths** with specific addressing.

## Recommendation

Start with **Solution 2 (Minimal Multicast)** - it's the simplest and should work like your empty builtin config but with more control over the multicast address.

The bottom line: Your complex ROS 2 system with controllers and MoveIt **needs multicast discovery** to function properly.