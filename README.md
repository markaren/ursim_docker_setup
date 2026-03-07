# Docker setup for ursim + ur_robot_driver

PS: Currently unable to connect to ROS node from outside of docker.

```bash
docker compose up --build
```
=======
Runs a UR5e simulation (URSim) alongside the `ur_robot_driver` ROS2 Jazzy node stack on Windows with Docker Desktop.

## Requirements

- Docker Desktop for Windows with **host networking enabled**:
  `Settings → General → Enable host networking` (experimental feature)

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

Wait a few seconds for URSim to boot and `ur_driver` to connect.

### Development shell

Open an interactive shell with full access to all ROS2 nodes and topics:

```bash
docker exec -it ros2_dev-1 bash
```

Inside the shell:

```bash
# First run: reset the daemon so it discovers live nodes
ros2 daemon stop
ros2 node list      # shows all 24 ur_driver nodes
ros2 topic list     # shows all ur_driver topics
ros2 topic echo /joint_states --once
```


## Networking architecture

- `ursim` runs on Docker bridge `ur_net` (172.22.0.2); its UR control ports are TCP-mapped to Windows `localhost`
- `ur_driver` uses `network_mode: host` (Docker Desktop host networking); connects to URSim via `localhost:30001` etc.
- `ros2_dev` uses `network_mode: host`; shares the same network namespace as `ur_driver``
