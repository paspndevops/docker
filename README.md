# Docker Fundamentals: A Practical Guide

This document provides a clear and concise explanation of common Docker commands, offering a practical guide to building, running, and managing your containerized applications.

-----

## 🐋 Core Concepts

Before diving into the commands, let's understand a few key Docker concepts:

  * **Image:** A lightweight, stand-alone, executable package that includes everything needed to run a piece of software, including the code, a runtime, libraries, environment variables, and config files.
  * **Container:** A running instance of an image. It is a sandboxed environment that runs on the host machine's kernel but is isolated from other containers and the host itself.
  * **Dockerfile:** A text document that contains all the commands a user could call on the command line to assemble an image.
  * **Docker Hub:** A cloud-based registry service that allows you to link to code repositories, build your images and test them, stores manually pushed images, and links to Docker Cloud so you can deploy images to your hosts.

-----

## 🛠️ Essential Commands Explained

Here's a breakdown of the commands you've provided, placed in a logical workflow.

### Building Your Image

First, you need a `Dockerfile` to define your image. The following command builds an image from a `Dockerfile`.

```bash
#docker build -t satya/ssh:0.0.1 . -f ssh-dockerfile
```

  * `docker build`: The command to build a Docker image.
  * `-t satya/ssh:0.0.1`: This flags tags the image with a name (`satya/ssh`) and a specific version (`0.0.1`). This makes it easy to identify and manage different versions of your image.
  * `.`: This specifies that the build context (the set of files to be used for the build) is the current directory.
  * `-f ssh-dockerfile`: This flag specifies the name of the `Dockerfile` to use. If your `Dockerfile` is named `Dockerfile` (the default), you can omit this flag.

### Managing Docker Networks

Docker containers can communicate with each other and the outside world through networks.

```bash
#docker network create --driver bridge my-network
```

  * `docker network create`: Creates a new Docker network.
  * `--driver bridge`: Specifies that the network should use the `bridge` driver, which is the default and most common network driver. Bridge networks allow containers on the same host to communicate with each other.
  * `my-network`: The name you are giving to your new network.

### Running Your Container

Once you have an image, you can run it as a container.

```bash
#docker run -d -p 32:22 --name ssh-svr --network some-network satya/ssh:0.0.1
```

  * `docker run`: The command to run a Docker container.
  * `-d`: Runs the container in detached mode (in the background).
  * `-p 32:22`: This publishes a port from the host to the container. It maps port `32` on the host to port `22` inside the container. This allows you to connect to the SSH server running on port `22` inside the container by accessing port `32` on your host machine.
  * `--name ssh-svr`: Assigns a custom name to your container for easy reference.
  * `--network some-network`: Connects the container to the specified network.
  * `satya/ssh:0.0.1`: The name and tag of the image to run.

### Configuring Your Firewall (UFW)

If you are using a firewall like UFW (Uncomplicated Firewall) on your host machine, you need to allow traffic to the port you exposed.

```bash
#sudo ufw allow proto tcp from any to any port 22
#sudo ufw allow proto tcp from any to any port 32
#sudo ufw enable
#sudo ufw status
```

  * `sudo ufw allow proto tcp from any to any port 22`: This command allows incoming TCP traffic to port `22`.
  * `sudo ufw allow proto tcp from any to any port 32`: This allows incoming TCP traffic to port `32`, which is the port we mapped to our container's SSH port.
  * `sudo ufw enable`: Enables the UFW firewall.
  * `sudo ufw status`: Checks the status of the UFW firewall, showing the configured rules.

### Interacting with Your Container

Here's how you can interact with the running container.

#### Checking Port Mappings

To confirm the port mappings for your container:

```bash
#docker port ssh-svr 22
```

This command will display the public-facing port on the host that is mapped to the container's internal port `22`.

#### Connecting via SSH

Now you can connect to the SSH server inside the container:

```bash
#ssh root@127.0.0.1 -p 32
```

  * `ssh root@127.0.0.1`: Attempts to connect to an SSH server as the `root` user on your local machine.
  * `-p 32`: Specifies that the SSH client should connect to port `32`, which is the port you mapped to the container.

#### Connecting to Other Containers

If you have other containers on the same network (e.g., a MySQL database container named `mysql-server`), your application container can connect to them using their container names as hostnames.

```bash
#mysql -h mysql-server -uguest -p
```

  * `mysql`: The MySQL client command.
  * `-h mysql-server`: Specifies the hostname of the MySQL server. Because the containers are on the same Docker network, you can use the container name `mysql-server` to resolve to the correct IP address.
  * `-uguest -p`: Specifies the username and prompts for a password.

-----

## 🧹 Housekeeping: Stopping and Cleaning Up

To manage your Docker environment effectively, it's important to stop and remove unused resources.

```bash
#sudo docker container stop $(sudo docker container ls -aq) && sudo docker system prune -af --volumes
```

This is a powerful command that performs two actions:

1.  `sudo docker container stop $(sudo docker container ls -aq)`:

      * `sudo docker container ls -aq`: This lists the IDs of all containers (running and stopped).
      * `sudo docker container stop ...`: This stops all the running containers identified by the previous command.

2.  `sudo docker system prune -af --volumes`:

      * `sudo docker system prune`: A command to clean up unused Docker resources.
      * `-a`: Removes all unused images, not just dangling ones.
      * `-f`: Forces the removal without prompting for confirmation.
      * `--volumes`: Removes all unused volumes. **Use this with caution**, as it will permanently delete any data stored in volumes that are not currently in use by a container.

-----

## 🔒 A Note on Security: `docker.sock` Permissions

The command `sudo chmod 666 /var/run/docker.sock` is often used to grant non-root users access to the Docker daemon. While this can be convenient for development, it is **highly insecure**.

The `/var/run/docker.sock` file is the Unix socket that the Docker daemon listens on. Giving it `666` permissions means that any user on the system can interact with the Docker daemon, effectively granting them root-level privileges on the host machine.

A more secure approach is to add your user to the `docker` group:

```bash
sudo usermod -aG docker ${USER}
```

After running this command, you will need to log out and log back in for the changes to take effect. This allows you to run `docker` commands without `sudo` and without the security risks of changing the socket permissions.
