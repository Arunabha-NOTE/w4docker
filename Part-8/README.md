# Docker Compose for Multi-Container Applications

Docker Compose is a tool for defining and running multi-container Docker applications. With Compose, a single YAML file configures all of the application's services, networks, and volumes, simplifying their management.

## 1. Prerequisites

Before using Docker Compose, ensure the following are in place:

- **Docker Engine:** Docker must be installed and running on the system.
- **Docker Compose:** Docker Compose typically comes pre-installed with Docker Desktop. For Linux, it might need to be installed separately. Verify its presence by running:

```bash
docker compose version
```

## 2. Project Structure

A typical multi-container application managed by Docker Compose involves a root directory containing a `docker-compose.yml` file, along with subdirectories for individual service code and their respective Dockerfiles.

```
my-multi-app/
├── web/
│   ├── Dockerfile
│   └── app.py
├── db/
│   └── init.sql
└── docker-compose.yml
```

## 3. Define Services in docker-compose.yml

The core of Docker Compose is the `docker-compose.yml` file. This YAML file declares the services that make up the application, along with their configurations.

Example `docker-compose.yml` for a simple web app with a database:

```yaml
# docker-compose.yml
version: '3.8' # Specify the Compose file format version

services:
  # Web service definition
  web:
    build: ./web # Instructs Docker Compose to build an image from the Dockerfile in the './web' directory
    ports:
      - "8000:5000" # Maps host port 8000 to container port 5000
    volumes:
      - ./web:/app # Mounts the local 'web' directory into the container at '/app' (for development)
    depends_on:
      - db # Ensures the 'db' service starts before the 'web' service
    environment:
      DATABASE_URL: postgres://user:password@db:5432/mydatabase # Environment variable for the web app to connect to the database

  # Database service definition
  db:
    image: postgres:13 # Uses the official PostgreSQL 13 image from Docker Hub
    environment:
      POSTGRES_DB: mydatabase
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
    volumes:
      - db_data:/var/lib/postgresql/data # Mounts a named volume for persistent database data
    
# Define volumes used by services
volumes:
  db_data: # Declares the named volume
```

### Explanation of common directives:

- **`version`:** Specifies the Compose file format version.
- **`services`:** Defines the individual containers (services) that make up the application.
- **`build`:** Specifies the path to the directory containing a Dockerfile to build an image for this service.
- **`image`:** Specifies an existing Docker image to use for the service.
- **`ports`:** Publishes container ports to the host.
- **`volumes`:** Mounts host paths or named volumes into the container for data persistence or code synchronization.
- **`environment`:** Sets environment variables inside the container.
- **`depends_on`:** Declares dependencies between services, ensuring they start in the correct order.
- **`networks`:** Connects a service to specific networks (Compose creates a default network if none are specified).
- **`volumes`:** Defines named volumes that can be used by services for persistent data.
- **`networks`:** Defines custom networks if needed, beyond the default bridge network created by Compose.

## 4. Basic Docker Compose Commands

Once the `docker-compose.yml` file is defined, the application can be managed with simple commands from the project's root directory.

### Build, create, and start all services:

```bash
docker compose up
```

### To run in detached mode (background):

```bash
docker compose up -d
```

### Build images (without starting containers):

```bash
docker compose build
```

### List running services/containers:

```bash
docker compose ps
```

### View logs from all services:

```bash
docker compose logs
```

### To view logs for a specific service (e.g., web):

```bash
docker compose logs web
```

### Stop running services (without removing containers):

```bash
docker compose stop
```

### Stop and remove all services, networks, and volumes defined in the Compose file:

```bash
docker compose down
```

### To remove volumes (including persistent data):

```bash
docker compose down --volumes
```

### Restart all services:

```bash
docker compose restart
```

### Execute a command in a service container:

```bash
docker compose exec web bash # Opens a bash shell in the 'web' service container
```

## 5. Scaling Services

Docker Compose can easily scale services to run multiple instances:

```bash
docker compose up --scale web=3 -d
```

This command starts three instances of the web service in detached mode.

---

# Docker Security Best Practices

Securing Docker environments involves practices across the host, images, and runtime. Adhering to these best practices significantly reduces the attack surface and mitigates potential vulnerabilities.

## 1. Image Security

### Use Trusted Base Images
Prioritize official Docker images or images from verified publishers on Docker Hub. For custom images, choose minimal base images (e.g., Alpine variants, scratch, distroless) to reduce the attack surface.

### Pin Image Versions
Always specify exact image tags (e.g., `node:18.17.0-alpine` instead of `node:latest`) in Dockerfiles and Compose files to ensure reproducible builds and avoid unexpected changes when the "latest" tag updates.

### Minimize Image Size
Use multi-stage builds to exclude build-time dependencies, SDKs, and unnecessary files from the final production image. A smaller image means fewer potential vulnerabilities.

### Leverage .dockerignore
Exclude sensitive files, source code not needed for runtime, and build artifacts from the build context using a `.dockerignore` file.

### Scan Images for Vulnerabilities
Integrate image scanning tools (e.g., Docker Scout, Trivy, Clair) into the CI/CD pipeline to identify and remediate known vulnerabilities before deployment.

### Avoid Storing Secrets in Images
Never hardcode API keys, passwords, or other sensitive information directly into Dockerfiles or images. Use Docker Secrets, environment variables (for development), or dedicated secret management solutions.

### Rebuild Images Regularly
Docker images are immutable. Rebuild images frequently to incorporate the latest security patches and updates from base images and dependencies.

## 2. Container Runtime Security

### Run as Non-Root User
By default, processes inside containers run as root. Define a non-root `USER` in the Dockerfile to run applications with least privilege, limiting potential damage in case of a container breakout.

### Limit Container Capabilities (--cap-drop, --cap-add)
Containers run with a default set of Linux capabilities. Drop unnecessary capabilities (e.g., `--cap-drop ALL`) and only add back those strictly required for the application's function. Avoid `--privileged` unless absolutely necessary.

### Set Read-Only Filesystems (--read-only)
For stateless applications, run containers with a read-only root filesystem to prevent malicious writes or tampering.

### Limit Resource Usage (--memory, --cpus)
Implement resource quotas to prevent a single compromised container from consuming excessive host resources, leading to a Denial-of-Service (DoS) for other containers or the host.

### Disable Privilege Escalation (--security-opt=no-new-privileges)
Prevent processes inside the container from gaining additional privileges.

### Use Security Profiles (AppArmor, SELinux, Seccomp)
Enable and configure kernel security modules to restrict container actions. Docker provides default Seccomp profiles; custom profiles can be created for stricter control.

### Do Not Expose Unnecessary Ports
Only expose ports that are absolutely required for the application to function.

### Avoid Running SSH Within Containers
Access containers via `docker exec` for debugging and interaction. Running an SSH daemon adds unnecessary complexity and attack surface.

### Implement Health Checks
Add `HEALTHCHECK` instructions in Dockerfiles to ensure containers are functioning correctly, allowing orchestration tools to replace unhealthy instances.

### Monitor Container Activity
Implement robust logging and monitoring solutions to detect unusual or malicious activity within containers.

## 3. Docker Daemon and Host Security

### Keep Docker Up-to-Date
Regularly update the Docker Engine and Docker Desktop to benefit from the latest security patches and bug fixes.

### Protect the Docker Daemon Socket
The Docker daemon socket (`/var/run/docker.sock`) is effectively root access to the Docker host. Never expose it directly to untrusted containers or over the network without proper authentication (TLS).

### Harden the Host OS
Apply security best practices to the underlying operating system running Docker (e.g., regular patching, minimal services, firewall rules).

### Run Docker in Rootless Mode
If feasible for the use case, configure Docker to run in rootless mode, which allows the Docker daemon and containers to run as a non-root user, significantly reducing the impact of a container escape.

### Implement Docker Content Trust (DCT)
Use DCT to verify the authenticity and integrity of images pulled from registries by checking digital signatures.

By diligently applying these practices, organizations can significantly enhance the security posture of their containerized applications and infrastructure.