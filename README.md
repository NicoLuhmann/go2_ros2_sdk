# go2_ros2_sdk — Headless 2D Nav Fork

Customized fork of [abizovnuralem/go2_ros2_sdk](https://github.com/abizovnuralem/go2_ros2_sdk) for headless deployment on a **Unitree Go2 EDU Jetson Nano** with 2D occupancy grid mapping, Nav2 AMCL navigation, and Foxglove bridge visualization.

## What changed from upstream

| Change | Reason |
|--------|--------|
| Removed `coco_detector` | Object detection not needed |
| Removed `speech_processor` | No audio output on headless robot |
| Removed `lidar_processor` (Python) | Replaced by `lidar_processor_cpp` — same pipeline, ~3.5× faster |
| Removed `docker/` | Project uses its own Docker setup at repo root |
| Removed `3d_map.ply` | Stale artifact from Python lidar processor |
| Removed `torch`, `torchvision`, `open3d`, `pydub` from `requirements.txt` | Dependencies of removed packages |
| RViz disabled by default in all launch files | Robot is headless; visualization via Foxglove |
| TTS nodes removed from all launch files | No speech processor |

## Packages

| Package | Description |
|---------|-------------|
| `go2_interfaces` | Custom ROS2 message definitions for Go2 communication |
| `go2_robot_sdk` | Main robot driver — WebRTC/CycloneDDS connection, IMU, joint states, camera |
| `lidar_processor_cpp` | C++ LiDAR pipeline: raw data → PointCloud2 → filtered cloud for SLAM/Nav2 |

## Quick start

Set environment variables (e.g. in `.env`):

```bash
ROBOT_IP=192.168.123.x   # Go2 IP on the internal LAN
ROBOT_TOKEN=             # Only needed for WebRTC — leave empty for CycloneDDS
CONN_TYPE=cyclonedds     # See note below
```

> **`CONN_TYPE` — robot connection, not ROS2 middleware.**
> This controls how `go2_driver_node` connects *to the robot*, not how ROS2 nodes talk to each other.
> The ROS2 middleware (also CycloneDDS) is configured automatically by the container via `RMW_IMPLEMENTATION=rmw_cyclonedds_cpp` and the mounted `cyclonedds.xml`.
>
> - Use `cyclonedds` if the container is on the same LAN as the Go2 (e.g. Jetson Nano plugged into the robot network). The driver subscribes to the Go2's native DDS topics directly — lower latency, no extra connection setup, and consistent with how Foxglove bridge already discovers topics in this setup.
> - Use `webrtc` only if connecting over WiFi to a different network or if the robot firmware requires it.

**Mapping** — drive the robot around to build a 2D occupancy map:

```bash
ros2 launch go2_robot_sdk mapping.launch.py
```

Save the map via the slam_toolbox service or RViz plugin, then use it for navigation.

**Navigation** — localize and navigate autonomously with a saved map:

```bash
ros2 launch go2_robot_sdk navigation.launch.py map:=/path/to/map.yaml
```

Both launches start Foxglove bridge on port `8765`. Connect from Foxglove Studio on your workstation.

## Dependencies

Handled by the project Dockerfile. Key ROS packages: `slam_toolbox`, `nav2_bringup`, `pointcloud_to_laserscan`, `pcl_ros`, `foxglove_bridge`.
