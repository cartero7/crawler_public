# ROS 2 Domain 2 Quickstart

This guide is for an operator who wants to connect a terminal to the crawler
ROS 2 system running on `ROS_DOMAIN_ID=2`.

It assumes:

- ROS 2 Jazzy is installed
- this workspace is already built
- the geographic simulator or the crawler stack is already running

## Prepare the Terminal

Run this in every new terminal:

```bash
source /opt/spaceapps/obsea_crawler/scripts/source_ros_env.sh
```

If the shared workspace has not been deployed to `/opt/spaceapps/obsea_crawler`
yet, deploy it first from the original checkout:

```bash
cd /home/carla/obsea_crawler
./scripts/deploy_spaceapps_workspace.sh
```

Check the domain:

```bash
printenv ROS_DOMAIN_ID
```

Expected value:

```text
2
```

If the domain does not match, ROS 2 CLI commands may return `Node not found`
even when the service is running correctly.

## Basic Checks

List nodes:

```bash
ros2 node list
```

For the geographic simulator, you should see at least:

```text
/crawler_geo_sim_node
```

List topics:

```bash
ros2 topic list
```

Check the current control mode:

```bash
ros2 param get /crawler_geo_sim_node control.mode
```

Check the current trajectory type:

```bash
ros2 param get /crawler_geo_sim_node trajectory.type
```

## Control Model

Two different parameters are involved:

- `control.mode`: how the node is controlled
- `trajectory.type`: which automatic path it follows

They are independent parameters. Do not put trajectory names into
`control.mode`.

Valid values for `control.mode`:

```text
trajectory
manual
```

Valid examples for `trajectory.type`:

```text
line
circle
slalom
square
patrol
```

These commands are invalid:

```bash
ros2 param set /crawler_geo_sim_node control.mode slalom
ros2 param set /crawler_geo_sim_node control.mode line
```

They fail because `slalom` and `line` are trajectory types, not control modes.

## Switch Modes

Set the node to automatic mode:

```bash
ros2 param set /crawler_geo_sim_node control.mode trajectory
```

Set manual mode:

```bash
ros2 param set /crawler_geo_sim_node control.mode manual
```

Setting `control.mode=trajectory` only switches to automatic control.
It does not change `trajectory.type`.

Example:

```bash
ros2 param set /crawler_geo_sim_node control.mode trajectory
ros2 param get /crawler_geo_sim_node trajectory.type
```

If `trajectory.type` was `line`, it stays `line` until you change it explicitly.

## Change Trajectory Type

Valid sequence:

```bash
ros2 param set /crawler_geo_sim_node control.mode trajectory
ros2 param set /crawler_geo_sim_node trajectory.type line
```

More valid examples:

```bash
ros2 param set /crawler_geo_sim_node trajectory.type circle
ros2 param set /crawler_geo_sim_node trajectory.type slalom
```

Available trajectory types:

```text
line, square, circle, eight, sweep, spiral, flower, slalom, patrol
```

## Manual Motion

Publish a forward command:

```bash
ros2 topic pub /crawler/motion/cmd crawler_interfaces/msg/MotionCommand "{left_command: 0.4, right_command: 0.4}" -r 5
```

Publish a turn command:

```bash
ros2 topic pub /crawler/motion/cmd crawler_interfaces/msg/MotionCommand "{left_command: 0.4, right_command: -0.4}" -r 5
```

Publish a stop command:

```bash
ros2 topic pub --once /crawler/motion/cmd crawler_interfaces/msg/MotionCommand "{left_command: 0.0, right_command: 0.0}"
```

Notes:

- a non-zero manual command switches the node to `manual`
- a `0.0 / 0.0` command does not force the mode change

## Common Problems

`Node not found`

- check `printenv ROS_DOMAIN_ID`
- make sure it is `2`
- run `ros2 node list`

Warnings about `ROS_LOCALHOST_ONLY`

- run `unset ROS_LOCALHOST_ONLY`

Foxglove can connect but no ROS data appears

- check that `crawler-geo-sim.service` is running
- check that port `9090` is listening

Useful commands:

```bash
sudo systemctl status crawler-geo-sim.service --no-pager
sudo journalctl -u crawler-geo-sim.service -b --no-pager | tail -n 100
ss -ltn sport = :9090
```
