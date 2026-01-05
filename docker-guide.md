# Docker Guide (Ubuntu)

This guide follows the official Docker Engine install docs for Ubuntu: https://docs.docker.com/engine/install/ubuntu/

## Install Docker Engine (Docker CE)

Remove potentially-conflicting packages (safe if not installed):

```bash
for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do
  sudo apt remove -y $pkg
done
```

Set up Docker’s APT repository and install:

```bash
sudo apt update
sudo apt install -y ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo \"$VERSION_CODENAME\") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

Start Docker and verify:

```bash
sudo systemctl enable --now docker
docker version
sudo docker run hello-world
```

Optional (run Docker without `sudo`):

```bash
sudo usermod -aG docker $USER
newgrp docker
docker run hello-world
```

If you still get “permission denied” on `/var/run/docker.sock`, log out and back in (group membership is applied at login).

## Common Docker commands

Basics:

```bash
docker version
docker info
docker ps            # running containers
docker ps -a         # all containers
docker images
```

Run and clean up:

```bash
docker run --rm hello-world
docker run --rm -it ubuntu:24.04 bash
docker run -d --name web -p 8080:80 nginx:alpine
```

Inspect and debug:

```bash
docker logs -f web
docker exec -it web sh
docker inspect web
docker port web
```

Stop/remove containers and images:

```bash
docker stop web
docker rm web
docker rmi nginx:alpine
```

Disk cleanup:

```bash
docker system df
docker system prune            # removes unused containers/images/networks
docker system prune -a         # also removes unused images not just dangling
docker volume prune            # removes unused volumes
```

## Docker Compose (recommended for multi-service projects)

Common workflow (from the directory with `compose.yaml` / `docker-compose.yml`):

```bash
docker compose up -d
docker compose ps
docker compose logs -f
docker compose exec <service> sh
docker compose down
```

Build and update:

```bash
docker compose pull
docker compose build
docker compose up -d --build
```

## Build images

```bash
docker build -t myapp:dev .
docker run --rm -p 8000:8000 myapp:dev
```

## Tag and push (registries)

```bash
docker login
docker tag myapp:dev ghcr.io/<org-or-user>/myapp:dev
docker push ghcr.io/<org-or-user>/myapp:dev
```

## Troubleshooting

- Docker service status: `sudo systemctl status docker` and logs: `journalctl -u docker --no-pager -n 200`
- “Cannot connect to the Docker daemon”: `sudo systemctl start docker`
- Proxy/corporate network issues: configure Docker daemon proxy + restart Docker (see Docker docs)
