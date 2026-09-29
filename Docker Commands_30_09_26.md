---

# Docker Flashcards

### 1. Container Lifecycle Management

* **`docker ps`**
* **Use:** Lists all currently running containers, showing IDs, image names, mapped ports, and container names.


* **`docker ps -a`**
* **Use:** Lists all containers, including both running and stopped (`Exited`) ones.


* **`docker stop <container>`**
* **Use:** Gracefully stops a running container by sending a termination signal (e.g., stopping `my-new-webappcontainer` or `volume-nginx`).


* **`docker start <container>`**
* **Use:** Starts an existing stopped container without re-creating it.


* **`docker rm <container>`**
* **Use:** Deletes a stopped container.


* **`docker rm -f <container_1> <container_2>`**
* **Use:** Forcefully stops and removes one or more containers in a single command (used to cleanup `ntwk-client` and `network-ngnix`).



---

### 2. Inspecting & Interacting with Containers

* **`docker logs <container>`**
* **Use:** Fetches and displays stdout/stderr logs from a container (used to view Nginx initialization and HTTP request logs).


* **`docker exec -it <container> sh`**
* **Use:** Opens an interactive shell session inside a running container to explore its filesystem or test commands interactively.


* **`docker exec <container> <command>`**
* **Use:** Executes a single command directly inside a running container without entering its shell (e.g., `docker exec volume-nginx cat /usr/share/nginx/html/index.html`).



---

### 3. Bind Mounts (Host Filesystem Mounting)

* **`docker run -d --name <name> -p <host_port>:<container_port> -v "${PWD}:/path/in/container" <image>`**
* **Use:** Runs a container in detached mode (`-d`), maps host ports (`-p`), and bind-mounts a local directory from your host machine (`${PWD}`) into the container filesystem (`-v`).
* **Key Takeaway:** Absolute paths starting with `/` are required for container destinations (e.g., `:/usr/share/nginx/html`).



---

### 4. Docker Named Volumes

* **`docker volume create <volume_name>`**
* **Use:** Creates a managed persistent storage volume (`mydata`) independent of the container lifecycle.


* **`docker volume ls`**
* **Use:** Lists all local Docker volumes.


* **`docker run -d --name <name> -p <host_port>:<container_port> -v <volume_name>:/path/in/container <image>`**
* **Use:** Mounts a Docker named volume to a directory inside a container (e.g., `-v mydata:/usr/share/nginx/html`), allowing persistent data sharing across containers.



---

### 5. Docker Custom Networking & Service Discovery

* **`docker network ls`**
* **Use:** Lists all available networks (default networks include `bridge`, `host`, and `none`).


* **`docker network create <network_name>`**
* **Use:** Creates a custom user-defined bridge network (`my-network`) that provides automatic DNS resolution between containers.


* **`docker run -d --name <name> --network <network_name> <image>`**
* **Use:** Attaches a new container directly to a custom network upon creation.


* **`docker run -it --name <name> --network <network_name> busybox sh`**
* **Use:** Launches an interactive temporary tool container (`busybox`) attached to your custom network for testing connectivity.


* **`wget -qO- http://<container_name>`**
* **Use:** Tests internal HTTP networking between containers on the same custom network using the container name as a hostname (DNS discovery).



---

### 6. Local Image Management

* **`docker images`**
* **Use:** Lists locally stored Docker images along with their IDs, disk usage, and size (`hello-world`, `my-web-app`, `nginx`).


* **`docker pull <image_name>`**
* **Use:** Downloads an image from Docker Hub to your local engine repository.



---

### Key Errors & Syntax Lessons Fixed

1. **`docker build` vs `docker run`:** Options like `-d`, `--name`, `-p`, and `-v` belong to `docker run`, not `docker build`.
2. **Mount Paths:** Relative paths fail in volume mounting; container target paths must be absolute (e.g., `/usr/share/nginx/html`).
3. **Volume Formatting:** Ensure no trailing spaces around colons in volume flags (`-v mydata:/path`, not `-v mydata: /path`).
4. **Port Allocation:** Only one container can bind to a specific host port at a time (e.g., Port `8080` vs `8081`).
5. **Image vs Flag Ordering:** In `docker run`, image options must come **before** the image name, and command options after (e.g., `docker run -it --name client --network my-network busybox sh`).