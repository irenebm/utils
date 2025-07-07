# cuMotion MoveIt Timeout Troubleshooting Guide

## Problem Summary

You're experiencing timeout issues with MoveIt when using cumotion for robot arm motion planning. The logs show:

- **nvblox ESDF integration delays**: 0.598 seconds
- **nvblox depth image integration delays**: 0.487 seconds  
- **MoveIt move_group timeout**: "Timed out!" error
- **Robot segmentation computational overhead**: 180-240ms processing times

## Root Cause Analysis

The timeout is occurring because:
1. nvblox is taking too long to update the collision geometry (ESDF)
2. cumotion depends on updated collision geometry from nvblox for safe motion planning
3. MoveIt's default timeout is being exceeded before planning can complete

## Solutions (in order of recommendation)

### 1. Increase MoveIt Timeout Parameters

**Immediate Fix**: Increase the timeout values in your MoveIt configuration.

Add/modify these parameters in your MoveIt configuration files:

```yaml
# In your kinematics.yaml file
<planning_group_name>:
  kinematics_solver: kdl_kinematics_plugin/KDLKinematicsPlugin
  kinematics_solver_timeout: 0.1  # Increase from default 0.05
  kinematics_solver_attempts: 5   # Increase from default 3

# In your move_group launch file or parameters
move_group:
  planning_time_limit: 10.0           # Default is usually 5.0
  max_planning_time: 15.0             # Increase max planning time
  allowed_execution_duration_scaling: 2.0  # Allow 2x expected execution time
  execution_duration_monitoring: true      # Keep monitoring enabled
```

### 2. Optimize nvblox Performance

**Configuration Changes**:

```yaml
# In your nvblox configuration file
nvblox_node:
  # Reduce ESDF update frequency
  esdf_2d_update_rate_hz: 4.0      # Default is often 10.0
  mesh_update_rate_hz: 5.0         # Default is often 10.0
  
  # Reduce integration rates  
  max_integration_time_s: 0.1      # Limit time spent on integration
  
  # Optimize voxel sizes (larger = faster)
  voxel_size: 0.05                 # Increase from default 0.02 if possible
  
  # Reduce truncation distance
  truncation_distance_vox: 4       # Reduce from default values
  
  # Enable performance optimizations
  use_depth_preprocessing: true
  use_color: false                 # Disable if not needed
```

### 3. Reduce Robot Segmentation Overhead

The robot_segmenter_node is consuming significant compute time (180-240ms). Optimize it:

```yaml
robot_segmenter_node:
  # Reduce processing frequency
  processing_rate_hz: 5.0          # Down from default higher rates
  
  # Reduce image resolution if possible
  depth_image_scale: 0.5           # Process at half resolution
  
  # Optimize segmentation parameters
  use_approximate_sync: true
  queue_size: 3                    # Reduce queue size
```

### 4. Adjust cuMotion Configuration

**Optimize cuMotion planning parameters**:

```yaml
cumotion:
  # Reduce planning time per attempt
  max_attempts: 3                  # Reduce number of planning attempts
  planning_timeout: 2.0            # Set explicit timeout for cumotion
  
  # Optimize collision checking
  collision_check_frequency: 0.1   # Reduce frequency of collision checks during planning
  
  # Use faster but less optimal planning
  use_fast_planning_mode: true     # If available in your cumotion version
```

### 5. System-Level Optimizations

**Hardware/System Tuning**:

```bash
# Increase GPU memory allocation for nvblox
export CUDA_VISIBLE_DEVICES=0
export CUDA_MPS_PIPE_DIRECTORY=/tmp/nvidia-mps
export CUDA_MPS_LOG_DIRECTORY=/tmp/nvidia-log

# Optimize CPU scheduling for real-time performance
sudo sysctl -w kernel.sched_rt_runtime_us=950000
sudo sysctl -w kernel.sched_rt_period_us=1000000

# Increase memory limits
ulimit -l unlimited
```

### 6. Launch File Optimizations

**Modify your launch file**:

```python
# In your robot launch file
def generate_launch_description():
    return LaunchDescription([
        # Set process priorities
        Node(
            package='nvblox_ros',
            executable='nvblox_node',
            name='nvblox_node',
            parameters=[nvblox_config],
            # Lower priority for nvblox to not block motion planning
            additional_env={'SCHED_PRIORITY': '10'}
        ),
        
        Node(
            package='moveit_ros_move_group',
            executable='move_group',
            name='move_group',
            parameters=[moveit_config],
            # Higher priority for motion planning
            additional_env={'SCHED_PRIORITY': '20'}
        ),
    ])
```

### 7. Alternative Collision Representations

**Consider these alternatives if the above don't work**:

```yaml
# Option A: Use simplified collision geometry
cumotion:
  use_mesh_collision: false        # Use primitive shapes instead
  collision_sphere_radius: 0.1     # Approximate robot with spheres
  
# Option B: Use pre-computed collision maps
nvblox:
  use_static_map: true            # If environment is mostly static
  map_update_frequency: 1.0       # Very low update rate
```

## Testing and Verification

### 1. Monitor Performance
```bash
# Monitor nvblox timing
ros2 topic echo /nvblox_node/timing_info

# Monitor MoveIt planning time  
ros2 topic echo /move_group/result

# Check system resources
htop
nvidia-smi -l 1
```

### 2. Incremental Testing
1. Start with timeout increases (Solution 1)
2. Add nvblox optimizations (Solution 2)  
3. Tune robot segmentation (Solution 3)
4. Apply system optimizations (Solution 5)

### 3. Success Metrics
- nvblox ESDF integration < 0.2s
- nvblox depth integration < 0.2s  
- No MoveIt timeout errors
- Successful motion planning completion

## Configuration File Locations

Common locations to modify:
```
/config/kinematics.yaml           # Kinematics timeouts
/config/move_group.yaml           # MoveIt planning parameters  
/config/nvblox_params.yaml        # nvblox configuration
/config/robot_segmentation.yaml   # Segmentation parameters
/launch/robot_bringup.launch.py   # Launch file modifications
```

## Emergency Fallback

If none of the above work, consider:
1. **Disable real-time collision checking**: Plan in static environment first
2. **Use MoveIt without cumotion**: Fall back to standard OMPL planners
3. **Pre-plan trajectories**: Generate trajectories offline when possible

## Additional Resources

- [MoveIt Timeout Configuration](https://moveit.picknik.ai/humble/doc/examples/kinematics_configuration/kinematics_configuration_tutorial.html)
- [nvblox Performance Tuning](https://github.com/nvidia-isaac/nvblox)
- [cuMotion Documentation](https://docs.omniverse.nvidia.com/isaacsim/latest/isaac_ros_packages/isaac_ros_cumotion.html)

---

**Next Steps**: 
1. Try Solution 1 (timeout increases) first as it's the quickest fix
2. Monitor the timing logs to see which optimizations have the most impact
3. Gradually apply other solutions based on your specific hardware constraints