# FastDDS Configuration Troubleshooting Analysis

## Problem Summary
FastDDS configuration works for `robot-viz` container but fails for `devkit` container, despite both mounting the same config file to `/WorkingData/fastdds.xml`.

## Key Differences Between Containers

### 1. **Different Base Images**
- `robot-viz`: Uses `${ROBOT_VIZ_IMAGE_NAME}`
- `devkit`: Uses `${DEVKIT_IMAGE_NAME}` with custom dockerfile `aivot-devkit.dockerfile`

**Potential Issue**: Different FastDDS versions or missing FastDDS installation in devkit image.

### 2. **Workspace Structure Differences**
- `robot-viz`: Mounts to `/workspaces/aivot_ws`
- `devkit`: Mounts to `/workspace/src/aivot/`

**Potential Issue**: Applications might be looking for config in different relative paths.

### 3. **Additional Build Arguments in Devkit**
```yaml
args:
  BASE_IMAGE: ${BASE_IMAGE_NAME}
  NUM_THREADS: ${NUM_THREADS}
  LIBREALSENSE_BACKEND: ${LIBREALSENSE_BACKEND}
  WITH_WARMUP: ${WITH_WARMUP}
```

## Most Likely Causes

### 1. **Environment Variable Not Set**
FastDDS needs the `FASTRTPS_DEFAULT_PROFILES_FILE` environment variable to locate the config:

```bash
export FASTRTPS_DEFAULT_PROFILES_FILE=/WorkingData/fastdds.xml
```

### 2. **Different FastDDS Installation**
The devkit dockerfile might be missing FastDDS installation or using an incompatible version.

### 3. **Working Directory Differences**
Applications in devkit might be running from a different working directory, affecting relative path resolution.

### 4. **Permission Issues**
The config file might not have proper read permissions in the devkit container.

## Recommended Solutions

### Solution 1: Verify Environment Variable
Add to your `aivot-common-config` or devkit-specific environment:
```yaml
environment:
  - FASTRTPS_DEFAULT_PROFILES_FILE=/WorkingData/fastdds.xml
```

### Solution 2: Debug Inside Container
Run these commands inside the devkit container:
```bash
# Check if file exists and is readable
ls -la /WorkingData/fastdds.xml

# Check environment variable
echo $FASTRTPS_DEFAULT_PROFILES_FILE

# Check FastDDS installation
ldconfig -p | grep fastrtps
```

### Solution 3: Explicit Path in Application
If using ROS 2, ensure your launch files or applications explicitly set the config path:
```xml
<param name="rmw_fastrtps_use_default_profiles" value="true"/>
```

### Solution 4: Alternative Mount Strategy
Try mounting to a more standard location:
```yaml
volumes:
  - ${HOME}/Project/aivot_setup/docker/fastdds.xml:/root/.ros/fastdds.xml
```

## Network Configuration Analysis

Your FastDDS config specifies:
- **Disabled multicast**: Good for containerized environments
- **Initial peers**: `192.168.56.192` and `192.168.56.200`
- **Socket buffers**: 12MB (good for high-throughput)

**Verification needed**: Ensure devkit container can reach the specified peer addresses.

## Next Steps

1. **Check environment variables** in both containers
2. **Verify FastDDS installation** in devkit image
3. **Test network connectivity** to peer addresses from devkit
4. **Compare working directories** and file permissions
5. **Enable FastDDS logging** to see detailed error messages

## FastDDS Logging Configuration

Add this to your config for debugging:
```xml
<log>
    <use_default>FALSE</use_default>
    <consumer>
        <class>StdoutConsumer</class>
    </consumer>
</log>
```

Or set environment variable:
```bash
export FASTRTPS_LOG_LEVEL=Log::Kind::Info
```