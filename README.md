# Unofficial Tiago Pro Simulation

## Quick Start

1. Clone this repo and start Docker:
   ```bash
   git clone <repo-url>
   cd tiago_pro/docker
   docker compose up
   ```

2. In the Docker container, launch the simulation:
   ```bash
   ros2 launch tiago_pro_gazebo tiago_pro_gazebo.launch.py is_public_sim:=True
   ```