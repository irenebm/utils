# FastDDS Devkit Container Issue Analysis

## Environment Variable Status ✅
The `FASTRTPS_DEFAULT_PROFILES_FILE=/WorkingData/fastdds.xml` is correctly set in `aivot-common-config`, so this is not the issue.

## Key Differences Between Working vs Non-Working Containers

### Working Containers (robot, robot-viz)
- **Workspace**: `/workspaces/aivot_ws` (plural)
- **Focused source mounting**: Only specific directories
- **Simpler builds**: Standard dockerfile

### Non-Working Container (devkit)
- **Workspace**: `/workspace` (singular) 
- **Broad source mounting**: Entire Aivot project via `${HOME}/Project/Aivot/:/workspace/src/aivot/`
- **Complex build**: Custom `aivot-devkit.dockerfile` with additional args

## Most Likely Root Causes

### 1. **Different Working Directory**
Applications in devkit might be running from `/workspace` vs `/workspaces/aivot_ws`, potentially affecting path resolution.

### 2. **Different FastDDS Installation in Devkit Dockerfile**
The `aivot-devkit.dockerfile` might:
- Install a different FastDDS version
- Use different RMW middleware configuration
- Missing FastDDS development packages
- Have conflicting FastDDS installations

### 3. **RMW Middleware Configuration**
Devkit might be configured to use a different RMW implementation that doesn't respect the FastDDS config.

## Debugging Steps

### Step 1: Compare FastDDS Installations
Run in both working and devkit containers:
```bash
# Check FastDDS version
fastrtps --version

# Check RMW implementation
echo $RMW_IMPLEMENTATION

# Check available RMW implementations
ros2 doctor --report | grep rmw
```

### Step 2: Verify Config Loading
In devkit container:
```bash
# Check if config is loaded
FASTRTPS_LOG_LEVEL=Log::Kind::Info ros2 run demo_nodes_cpp talker

# Look for: "Found profile" in the logs
```

### Step 3: Check Application Working Directory
```bash
# In devkit container
pwd
ls -la /WorkingData/fastdds.xml

# Compare with working container working directory
```

## Quick Fixes to Try

### Fix 1: Force RMW Implementation
Add to devkit environment:
```yaml
environment:
  - RMW_IMPLEMENTATION=rmw_fastrtps_cpp
```

### Fix 2: Alternative Config Mount
Try mounting to ROS standard location:
```yaml
volumes:
  - ${HOME}/Project/aivot_setup/docker/fastdds.xml:/opt/ros/humble/share/rmw_fastrtps_cpp/xml/DEFAULT_FASTRTPS_PROFILES.xml
```

### Fix 3: Explicit Config in Launch Files
If using ROS 2 launch files, add:
```xml
<param name="rmw_fastrtps_use_default_profiles" value="true"/>
```

## Investigation Priority

1. **Check RMW implementation** in devkit vs working containers
2. **Examine aivot-devkit.dockerfile** for FastDDS-related differences  
3. **Compare working directories** and relative path handling
4. **Verify FastDDS installation consistency** across images

## Expected Findings

The issue is most likely in the `aivot-devkit.dockerfile` having:
- Different RMW middleware configuration
- Incompatible FastDDS version
- Missing FastDDS development packages
- Different default RMW implementation

## Next Action
Examine the `aivot-devkit.dockerfile` to compare FastDDS/RMW installation with working containers.