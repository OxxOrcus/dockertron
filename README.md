# Docker Command Quickstart Guide

Welcome! This guide provides a walkthrough of essential Docker commands for managing containers and images. It covers everything from running your first container to deploying a service like MySQL. Each section includes explanations, tips, and how to perform the same actions using the Docker Desktop graphical user interface (GUI).

## Table of Contents

1.  [Running & Managing a Basic Container]
2.  [Interacting with a Running Container]
3.  [Naming and Deleting Resources]
4.  [Copying Files To/From a Container]
5.  [Example: Running a MySQL Database]
6.  [Example: Running a Web Server with Docker Compose]

-----

## 1. Running & Managing a Basic Container

This section covers the basics of pulling a Docker image and running it as a container.

### Command Line (CLI) ⌨️

```bash
# Pull the latest Ubuntu image from Docker Hub
docker pull ubuntu

# Run a container from the image. It will start and immediately stop
# because it has no long-running command to execute.
docker run ubuntu

# Run a container that executes a command (sleep for 10s) and then stops.
docker run ubuntu sleep 10

# To stop a long-running container, first find its ID.
# This will list only currently running containers.
docker ps

# Now, use the ID to stop it.
docker stop [container_id]

# To see all options for the 'run' command
docker run --help
```

### Description

  * **`docker pull ubuntu`**: Downloads the `ubuntu` image from Docker Hub, the official repository for Docker images.
  * **`docker run ubuntu`**: Creates and starts a new container from the `ubuntu` image. By default, it runs the image's default command. For the base Ubuntu image, this is just starting a shell, which exits immediately if not given anything to do.
  * **`docker stop [container_id]`**: Gracefully stops a running container. You can get the container ID by running `docker ps`.

### Tips ✨

  * To see **all** containers, including stopped ones, use the command `docker ps -a`.
  * You only need to use the first few unique characters of a container ID, not the entire string (e.g., `docker stop a4d` instead of `docker stop a4d2cce1b423`).

### Docker Desktop GUI 🖱️

1.  **Pulling an Image**:
      * Open Docker Desktop.
      * In the search bar at the top, type `ubuntu` and press Enter.
      * Find the official `ubuntu` image in the results and click the **Pull** button.
2.  **Running a Container**:
      * Go to the **Images** tab on the left.
      * Find your local `ubuntu` image.
      * Click the **Run** ▶️ button next to it.
      * An "Optional settings" dialog will appear. You can just click **Run** to start a basic container.
3.  **Stopping a Container**:
      * Go to the **Containers** tab.
      * Find your running container in the list.
      * Click the **Stop** ⏹️ button.

-----

## 2. Interacting with a Running Container

Most of the time, you'll want to run a container in the background and then "enter" it to run commands.

### Command Line (CLI) ⌨️

```bash
# Run a new Ubuntu container in detached (-d) and interactive (-it) mode.
docker run -dti ubuntu

# To interact with it, get the container ID or name
docker ps

# Use 'exec' to run a command inside the already running container.
# Here, we open a bash shell to interact with it.
docker exec -it [container_id_or_name] /bin/bash
```

### Description

  * **`docker run -dti ubuntu`**: This is a common combination.
      * `-d` (detached): Runs the container in the background.
      * `-t` (tty): Allocates a pseudo-TTY, which is what a terminal connects to.
      * `-i` (interactive): Keeps STDIN open, allowing you to type into the container.
  * **`docker exec -it ... /bin/bash`**: The `exec` command lets you execute a command *inside an already running container*. This is the standard way to get a shell inside a background container.

### Tips ✨

  * **`run` vs. `exec`**: Remember, `docker run` **creates a new container** every time. `docker exec` works on an **existing, running container**.
  * Once inside the container with `bash`, you can run any Linux commands (like `ls`, `cd`, `apt-get update`). Type `exit` to leave the container's shell without stopping the container itself.

### Docker Desktop GUI 🖱️

1.  Go to the **Containers** tab.
2.  Find the container you want to interact with.
3.  Hover over it and click the **Open in terminal** (CLI) icon Terminal. This will open a terminal session directly inside your container.

-----

## 3. Naming and Deleting Resources

Managing containers and images by name is much easier than using long IDs.

### Command Line (CLI) ⌨️

```bash
# Run a detached container and give it a custom name 'Ubuntu-A'
docker run -dti --name Ubuntu-A ubuntu

# You can now use the name instead of the ID to stop it
docker stop Ubuntu-A

# Remove a stopped container
docker rm Ubuntu-A

# Remove a Docker image (it cannot be in use by any container)
docker rmi ubuntu
```

### Description

  * **`--name Ubuntu-A`**: Assigns a human-readable name to your container. Names must be unique.
  * **`docker rm [name]`**: Deletes a **stopped** container. You cannot delete a running container unless you force it.
  * **`docker rmi [image]`**: Deletes an image from your local machine. You must first delete all containers that were created from this image.

### Tips ✨

  * To stop and remove a container in one go, you can force the removal of a running container: `docker rm -f Ubuntu-A`.
  * To clean up all stopped containers, run: `docker container prune`.

### Docker Desktop GUI 🖱️

1.  **Naming**: When you click the **Run** button from the Images tab, you can set a name in the "Optional settings" dialog.
2.  **Deleting a Container**: In the **Containers** tab, click the **Delete** 🗑️ icon next to the container you want to remove.
3.  **Deleting an Image**: In the **Images** tab, click the **Delete** 🗑️ icon next to the image you want to remove.

-----

## 4. Copying Files To/From a Container

You can easily move files between your host machine and a container.

### Command Line (CLI) ⌨️

```bash
# On your host machine, create a sample file
echo "Hello from host" > Arquivo.txt

# Copy the file from your host machine INTO the container
# Syntax: docker cp [host_path] [container_name]:[container_path]
docker cp Arquivo.txt Ubuntu-A:/aula/

# Copy a file FROM the container back to your host machine
# Syntax: docker cp [container_name]:[container_path] [host_path]
docker cp Ubuntu-A:/destino/Meuzip.zip ./Zipcopia.zip
```

### Description

  * **`docker cp`**: This command works like the `scp` command in Linux. It allows you to copy files in either direction. You need to specify the source path and the destination path. The container path is prefixed with the container's name or ID and a colon (`:`).

### Tips ✨

  * For persistent data or sharing files regularly (like source code or databases), using **Docker Volumes** is the recommended best practice instead of `docker cp`. Volumes mount a directory from your host machine into the container.

### Docker Desktop GUI 🖱️

File copying is best handled via the command line. You can use the terminal built into Docker Desktop (or any terminal on your computer) to run the `docker cp` commands. There is no direct "drag-and-drop" file-copying feature in the main GUI.

-----

## 5. Example: Running a MySQL Database

Let's put it all together by running a persistent service like MySQL and exposing it to our host machine.

### Command Line (CLI) ⌨️

```bash
# Pull the official MySQL image
docker pull mysql

# Run the MySQL container
docker run --name mysql-A -e MYSQL_ROOT_PASSWORD=Senha123 -p 3306:3306 -d mysql

# Access the container's internal shell
docker exec -it mysql-A bash

# Inside the container, connect to the MySQL server
mysql -u root -p

# (Enter 'Senha123' when prompted for the password)

# Once in MySQL, you can run SQL commands
# CREATE DATABASE aula;
# show databases;
# exit

# To get detailed information about the container, like its IP address
docker inspect mysql-A
```

### Description

  * **`-e MYSQL_ROOT_PASSWORD=Senha123`**: Sets an **e**nvironment variable inside the container. The MySQL image requires this variable to set the root user's password on first run.
  * **`-p 3306:3306`**: **P**ublishes or "maps" a port. This command maps port `3306` on your host machine to port `3306` inside the container. This is what allows you to connect to the MySQL database from your own computer using a database client. The format is `[host_port]:[container_port]`.
  * **`-d mysql`**: Runs the `mysql` image in **d**etached mode.

### Tips ✨

  * **Data Persistence is CRUCIAL**: The command above stores the database files *inside* the container. If you run `docker rm mysql-A`, **all your data will be lost forever!**
  * **The Correct Way (with Volumes)**: To save your data permanently, you should use a Docker Volume. This stores the database files on your host machine.
    ```bash
    # The -v flag mounts a volume
    docker run --name mysql-A -v mysql_data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=Senha123 -p 3306:3306 -d mysql
    ```
    Now, even if you delete the container, the data persists in the `mysql_data` volume.

### Docker Desktop GUI 🖱️

1.  Go to the **Images** tab, find the `mysql` image, and click **Run**.
2.  In the "Optional settings" dialog:
      * **Container Name**: Give it a name, like `mysql-A`.
      * **Ports**: In the "Host port" field, type `3306`. The container port will likely default to `3306`.
      * **Environment Variables**: Click to add a variable.
          * **Key**: `MYSQL_ROOT_PASSWORD`
          * **Value**: `Senha123`
      * **Volumes (Recommended)**: To save your data, map a volume. You can create a named volume or bind-mount a folder from your computer to `/var/lib/mysql` in the container.
3.  Click **Run**. Your MySQL container will start. You can now connect to it from any database tool on your computer at `localhost:3306`.

-----

## 6. Example: Running a Web Server with Docker Compose

Docker Compose is a tool for defining and running multi-container Docker applications. With a single command, you can start all the services defined in a `docker-compose.yml` file. This project includes a pre-configured file to run a simple Apache web server.

### Command Line (CLI) ⌨️

```bash
# Start the Apache web server in the background.
# This command will automatically pull the httpd image if you don't have it.
sudo docker compose up -d

# To check the status of your running services
sudo docker compose ps

# To view the logs from the web server
sudo docker compose logs

# To stop and remove the containers defined in the compose file
sudo docker compose down
```

### Description

  * **`docker compose up -d`**: This command reads the `docker-compose.yml` file in the current directory, creates the defined services (in this case, an Apache server), and runs them in detached mode (`-d`).
  * The `docker-compose.yml` file in this repository is configured to:
      * Use the latest `httpd` (Apache) image.
      * Map port 80 on your host machine to port 80 in the container, so you can access the website at `http://localhost`.
      * Mount the local `./website` directory into the container's web root, so any changes you make to the files are reflected live.

### Tips ✨

  * **Accessing the Website**: Once the container is running, open a web browser and navigate to `http://localhost`. You should see the sample web page.
  * **No `sudo`?**: If you encounter permission errors when running `docker` commands, it's common to prefix them with `sudo`. Depending on your system's Docker setup, you might be able to run them without `sudo` if your user is part of the `docker` group.
