# Devkit Container FastDDS Issue

## Problem Status
- ✅ **robot-viz**: talker ↔ listener works (intra-container)
- ❌ **devkit**: talker ↔ listener fails (intra-container)
- ✅ **FastDDS config**: correctly loaded in both containers

## Root Cause
This is **NOT** a network/IP issue - it's a **devkit-specific FastDDS installation problem**.

## Most Likely Causes

### 1. **Different RMW Implementation**
Devkit might be using a different RMW middleware:

**Check in both containers:**
```bash
# In robot-viz container
echo $RMW_IMPLEMENTATION
ros2 doctor --report | grep rmw

# In devkit container  
echo $RMW_IMPLEMENTATION
ros2 doctor --report | grep rmw
```

### 2. **Missing FastDDS Dependencies**
Devkit dockerfile might be missing FastDDS runtime libraries:

**Check in devkit container:**
```bash
# Check FastDDS installation
ldconfig -p | grep fastrtps
ls -la /opt/ros/*/lib/lib*fastrtps*

# Check RMW FastDDS
ls -la /opt/ros/*/lib/lib*rmw_fastrtps*
```

### 3. **Different FastDDS Version**
**Check versions in both containers:**
```bash
# Check FastDDS version
apt list --installed | grep fastrtps
dpkg -l | grep fastrtps
```

### 4. **Different Domain Participant Configuration**
**Check domain setup:**
```bash
echo $ROS_DOMAIN_ID
echo $FASTRTPS_DEFAULT_PROFILES_FILE
ls -la /WorkingData/fastdds.xml
```

### 5. **Port Conflicts in Devkit**
**Check if FastDDS ports are available:**
```bash
# Check if ports 7400-7450 are available
netstat -tulpn | grep 740
ss -tulpn | grep 740
```

## Debugging Steps

### Step 1: Compare RMW Implementations
```bash
# Robot-viz container
docker exec -it aivot-robot-viz-1 bash -c "ros2 doctor --report | grep rmw"

# Devkit container
docker exec -it aivot-devkit-1 bash -c "ros2 doctor --report | grep rmw"
```

### Step 2: Enable Detailed FastDDS Logging
**In devkit container:**
```bash
export FASTRTPS_LOG_LEVEL=Log::Kind::Info
export FASTRTPS_LOG_VERBOSITY=VERB_INFO
ros2 run demo_nodes_cpp talker
```

Look for error messages about:
- Profile loading failures
- Port binding errors
- Participant creation failures

### Step 3: Check Dockerfile Differences
Compare the FastDDS installation between:
- `aivot-devkit.dockerfile` 
- Robot-viz dockerfile (or base dockerfile)

### Step 4: Force RMW Implementation
**Try forcing FastDDS in devkit:**
```bash
export RMW_IMPLEMENTATION=rmw_fastrtps_cpp
ros2 run demo_nodes_cpp talker
```

## Expected Findings

The `aivot-devkit.dockerfile` likely has:
1. **Different RMW middleware** (e.g., using Cyclone DDS instead of FastDDS)
2. **Missing FastDDS packages** during build
3. **Conflicting FastDDS installation** 
4. **Different default RMW implementation**

## Quick Fixes to Try

### Fix 1: Force FastDDS RMW
Add to devkit environment:
```yaml
environment:
  - RMW_IMPLEMENTATION=rmw_fastrtps_cpp
```

### Fix 2: Install Missing FastDDS Packages
In devkit dockerfile, ensure:
```dockerfile
RUN apt-get update && apt-get install -y \
    ros-${ROS_DISTRO}-rmw-fastrtps-cpp \
    ros-${ROS_DISTRO}-fastrtps
```

### Fix 3: Check Default RMW
In devkit dockerfile:
```dockerfile
ENV RMW_IMPLEMENTATION=rmw_fastrtps_cpp
```

The issue is almost certainly in the `aivot-devkit.dockerfile` having a different FastDDS/RMW setup compared to the working containers.