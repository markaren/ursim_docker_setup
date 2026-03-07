# Docker setup for ursim + ur_robot_driver

> PS: Currently unable to connect to ROS node from outside of docker.

=======
This setup runs URSim alongside the `ur_robot_driver`. This allows commanding a simulated UR robot using ROS2.

## Requirements

- [Docker Desktop for Windows}(https://www.docker.com/products/docker-desktop/)
- [vcxsrv](https://github.com/marchaesen/vcxsrv) 

## Services

| Service | Description |
|---------|-------------|
| `ursim` | Universal Robots simulator, accessible at `http://localhost:6080` |
| `ur_driver` | ROS2 Jazzy `ur_robot_driver` connected to URSim |
| `ros2_dev` | Interactive ROS2 development shell |

## Usage

```bash
docker compose up --build
```

Wait a few seconds for URSim to boot and `ur_driver` to connect. The first build might take some time.

### URsim configuration

User interface: http://localhost:6080/vnc.html

1. Start robot from lower left menu option and exit menu.
2. Navigate to Innstallation ->  URCaps -> External Control. Insert 172.20.0.3 as Host IP.
3. Navigate to Program. Click URCaps/ External Control.
4. Navigate to Run. Press play.


### Development shell

Open an interactive shell with full access to all ROS2 nodes and topics:

```bash
docker exec -it docker-ros2_dev-1 bash
```

Inside the shell:

```bash
# First run: reset the daemon so it discovers live nodes
ros2 daemon stop
ros2 node list      # shows all ur_driver nodes
ros2 topic list     # shows all ur_driver topics
ros2 topic echo /joint_states --once  # test connectivity
```

### Test joint_trajectory_controller 

First move the robot to HOME position within URsim.
```bash
ros2 launch ur_robot_driver test_joint_trajectory_controller.launch.py
```

### Using MoveIt

> Note: Run XLaunch on Windows before trying to run GUI apps within the container.

```bash
ros2 launch ur_moveit_config ur_moveit.launch.py ur_type:=ur5e launch_rviz:=true
```

## Networking architecture

- `ursim` runs on Docker bridge `ur_net` (172.22.0.2); its UR control ports are TCP-mapped to Windows `localhost`
- `ur_driver` uses `network_mode: host` (Docker Desktop host networking); connects to URSim via `localhost:30001` etc.
- `ros2_dev` uses `network_mode: host`; shares the same network namespace as `ur_driver``
