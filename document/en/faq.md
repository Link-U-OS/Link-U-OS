# FAQ

## How much disk space does the aimde development environment require?
The aimde image size is approximately 18GB. Considering temporary files during installation and space needed for subsequent development, it is recommended to have at least 30GB of available disk space.

## Docker commands require sudo to execute
### Execution fails for regular users
```
  $ docker ps
  permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock

  # Using sudo executes successfully
  $ sudo docker ps
  CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```
### Root cause
The current user is not in the docker user group and does not have permission to access the Docker daemon socket. The Docker daemon binds to a Unix socket (/var/run/docker.sock) by default, which is owned by the root user and the docker group.
### Solution:
```
# Step 1: Create the docker user group (if it doesn't exist)
sudo groupadd docker

# Step 2: Add the current user to the docker group
sudo usermod -aG docker $USER

# Step 3: Restart the system
sudo reboot
```

## System cannot find Docker service
### Description
```
sudo systemctl start docker
Failed to start docker.service: Unit docker.service not found
```

### Solution
```
sudo apt update
sudo apt install docker.io
sudo systemctl start docker
sudo systemctl enable docker
```