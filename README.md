# Docker setup for ursim + ur_robot_driver

Runs a UR5e simulation (URSim) alongside the `ur_robot_driver` ROS2 Jazzy node stack on Windows with Docker Desktop.

## Requirements

- Docker Desktop for Windows with **host networking enabled**:
  `Settings → General → Enable host networking` (experimental feature)
- Windows host IP: `192.168.1.53` (update `cyclone_dds.xml` and `entrypoint.sh` if different)

## Services

| Service | Description |
|---------|-------------|
| `ursim` | Universal Robots simulator (UR5), accessible at `http://localhost:6080` |
| `ur_driver` | ROS2 Jazzy `ur_robot_driver` connected to URSim |
| `ros2_dev` | Interactive ROS2 development shell (same image as `ur_driver`) |

## Usage

```bash
docker compose up -d
```

Wait ~60 seconds for URSim to boot and `ur_driver` to connect.

### Development shell

Open an interactive shell with full access to all ROS2 nodes and topics:

```bash
docker exec -it condescending-mestorf-ros2_dev-1 bash
```

Inside the shell:

```bash
# First run: reset the daemon so it discovers live nodes
ros2 daemon stop
ros2 node list      # shows all 24 ur_driver nodes
ros2 topic list     # shows all ur_driver topics
ros2 topic echo /joint_states
```

### Verify ur_driver is connected

```bash
docker logs condescending-mestorf-ur_driver-1 --tail 20
# Look for: "Robot requested program" and "Sent program to robot"
```

## Networking architecture

- `ursim` runs on Docker bridge `ur_net` (172.22.0.2); its UR control ports are TCP-mapped to Windows `localhost`
- `ur_driver` uses `network_mode: host` (Docker Desktop host networking); connects to URSim via `localhost:30001` etc.
- `ros2_dev` uses `network_mode: host`; shares the same network namespace as `ur_driver`
- CycloneDDS uses unicast discovery (multicast disabled); peers: `192.168.1.53` + `127.0.0.1`

**Note on Windows-native ROS2 (pixi/RoboStack):** Docker Desktop on Windows puts host-networked containers on the Docker VM's internal network (`192.168.65.x`), not on the Windows network stack. Bidirectional DDS UDP between a Windows-native ROS2 client and the containers is not reliably achievable. Use the `ros2_dev` container instead.
