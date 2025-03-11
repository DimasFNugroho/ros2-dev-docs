# ROS2 Docker Development Environment with MkDocs

## Overview
This guide covers how to set up a containerized ROS2 development environment using Docker, document it with MkDocs, and deploy the documentation to GitHub Pages.

---

## Step 1: Install Docker
1. Install Docker from the official site:
   - [https://docs.docker.com/get-docker/](https://docs.docker.com/get-docker/)

2. Verify installation:
```bash
docker --version
```

3. Start Docker:
```bash
sudo systemctl start docker
```

4. Enable Docker to start on boot:
```bash
sudo systemctl enable docker
```

---

## Step 2: Set Up a ROS2 Docker Container

### 2.1. Create a Dockerfile
Create a new project folder and add a `Dockerfile`:
```bash
mkdir ros2_dev
cd ros2_dev
nano Dockerfile
```

Example `Dockerfile`:
```Dockerfile
FROM osrf/ros:galactic-desktop

ENV LANG=C.UTF-8
ENV LC_ALL=C.UTF-8
ENV ROS_DISTRO=galactic

RUN apt-get update && apt-get install -y \
    python3-pip \
    python3-colcon-common-extensions \
    build-essential \
    ros-$ROS_DISTRO-rclpy \
    ros-$ROS_DISTRO-demo-nodes-cpp

RUN mkdir -p /ros2_ws/src
WORKDIR /ros2_ws
RUN git clone https://github.com/ros2/demos src/demos
RUN source /opt/ros/$ROS_DISTRO/setup.bash && colcon build

CMD ["bash", "-c", "source /opt/ros/$ROS_DISTRO/setup.bash && bash"]
```

### 2.2. Build the Docker Container
```bash
docker build -t ros2-dev .
```

### 2.3. Run the Docker Container
```bash
docker run --name ros2_container -it --rm --net=host --privileged \
    -v /dev:/dev \
    -v $(pwd):/ros2_ws \
    ros2-dev
```

### 2.4. Fix Container Name Conflicts
Check for existing containers:
```bash
docker ps -a
```

If the container name already exists:
- Remove the container:
```bash
docker rm ros2_container
```
- Or rename it:
```bash
docker rename ros2_container ros2_container_old
```
- Or restart it:
```bash
docker start -ai ros2_container
```

### 2.5. Test the ROS2 Setup
In the container:
```bash
source /opt/ros/galactic/setup.bash
ros2 run demo_nodes_cpp talker
```

Open a new terminal and attach to the container:
```bash
docker exec -it ros2_container bash
source /opt/ros/galactic/setup.bash
ros2 run demo_nodes_cpp listener
```

---

## Step 3: Create an MkDocs Project

### 3.1. Install MkDocs
```bash
pip install mkdocs
```

### 3.2. Create a New MkDocs Project
```bash
mkdocs new docs
cd docs
```

### 3.3. Edit `index.md`
```markdown
# ROS2 Docker Setup Guide

Welcome to the ROS2 Docker Setup Guide.
```

### 3.4. Test the MkDocs Site Locally
```bash
mkdocs serve
```
Open your browser to:
- [http://127.0.0.1:8000](http://127.0.0.1:8000)

---

## Step 4: Push to GitHub

### 4.1. Add GitHub Remote Using Personal Access Token
1. Create a Personal Access Token in GitHub
2. Set remote:
```bash
git remote add origin https://<your-token>@github.com/your-username/ros2-dev-docs.git
```
3. Push to GitHub:
```bash
git push -u origin main
```

### 4.2. (Optional) Use SSH Instead of HTTPS
1. Create SSH key:
```bash
ssh-keygen -t ed25519
```
2. Add to GitHub  Settings  SSH and GPG Keys
3. Set remote:
```bash
git remote set-url origin git@github.com:your-username/ros2-dev-docs.git
```

---

## Step 5: Deploy with GitHub Pages
1. Deploy site:
```bash
mkdocs gh-deploy
```
2. Enable GitHub Pages in repo settings.

Site will be available at:
```text
https://your-username.github.io/ros2-dev-docs
```

---

##  Summary Checklist
 Docker container with ROS2    
 MkDocs site    
 GitHub authentication    
 GitHub Pages deployment    

---

##  Done! 
