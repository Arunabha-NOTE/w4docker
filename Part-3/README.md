# Docker Registry, DockerHub, Create a Multi-Stage Build

# What is a Docker Registry?
A Docker Registry is a storage and distribution system for Docker images. Think of it as a server or service that acts as a central library for container images.  

## Key characteristics of a Docker Registry:

Storage: It stores Docker images, which are essentially templates for creating containers.  
Distribution: It allows users to "pull" (download) images to their local machines and "push" (upload) images from their local machines.  
Organization: Images are typically organized into repositories, where a repository holds all the different versions (identified by tags) of a specific image (e.g., ubuntu is a repository, and ubuntu:latest, ubuntu:22.04, ubuntu:20.04 are different tags within that repository).  
Open Source: The core Docker Registry software is open-source. This means anyone can download and run it to set up their own private registry on their own servers or infrastructure.  
Types:  
Public Registries: Accessible by anyone (e.g., Docker Hub, Google Container Registry, Amazon ECR, Azure Container Registry).  
Private Registries: Restricted access, typically used by organizations to store proprietary or sensitive images. These can be hosted on-premises or as a cloud service.  
In essence, a Docker Registry provides the fundamental functionality for storing and sharing Docker images.  

## What is Docker Hub?  
Docker Hub is Docker Inc.'s official cloud-based Docker Registry. It is the largest and most widely used public registry for Docker images.  

Think of Docker Hub as a specific, very popular instance of a Docker Registry, hosted and managed by Docker itself.  

## Key features and characteristics of Docker Hub:  

Public and Private Repositories: It hosts a vast collection of public images (including "Official Images" from Docker and "Verified Publisher" images) that anyone can pull. It also offers private repositories (some free, more with paid plans) for users and teams to store their own images securely.  
Default Registry: When you install Docker, it's configured by default to pull images from and push images to Docker Hub if no other registry is specified.  
Community and Collaboration: It's a central place for the Docker community to share and discover images. It supports teams, organizations, and access control.  
Automated Builds: Docker Hub can integrate with source code repositories (like GitHub, Bitbucket) to automatically build new Docker images whenever changes are pushed to your code.  
Webhooks: It allows you to trigger actions (e.g., deploying an application) after an image is successfully pushed to a repository.  
Image Scanning: Provides features for scanning images for known vulnerabilities (often a paid feature).  
Usage Tracking: Offers basic statistics on image pulls and storage.  

# Multi-Stage Builds in Docker

Multi-stage builds are a powerful feature in Docker that allows you to create smaller, more efficient, and more secure Docker images. Before multi-stage builds, developers often had to use workarounds like building the application in one container and then copying the artifacts to another, or including build tools in the final production image.

## Why Multi-Stage Builds?

* **Smaller Image Sizes:** By separating the build environment from the runtime environment, you eliminate unnecessary build tools, SDKs, and intermediate files from your final image. This significantly reduces the image size, leading to faster pulls, pushes, and deployments.
* **Improved Security:** A smaller image means a smaller attack surface. Less software in the final image translates to fewer potential vulnerabilities.
* **Faster Builds (Caching):** Docker can cache individual stages. If a stage hasn't changed, Docker can reuse the cached layer, speeding up subsequent builds.
* **Cleaner Separation of Concerns:** Clearly separates the dependencies and steps required for building an application from those needed to run it.

## How Multi-Stage Builds Work

A multi-stage build consists of multiple `FROM` instructions in a single `Dockerfile`. Each `FROM` instruction starts a new build stage. You can selectively copy artifacts from one stage to another using the `COPY --from=<stage_name_or_index>` instruction.

The key idea is that the *final* image is only built from the *last* stage, and it only includes the necessary runtime artifacts copied from previous stages.

## Example: Multi-Stage Build for a Go Application

Let's illustrate with a common use case: building a Go application. Go applications compile into a single static binary, making them excellent candidates for multi-stage builds where the build environment (Go SDK) is entirely separate from the lightweight runtime environment (e.g., `scratch` or `alpine`).

### Project Structure:
```
go-app/
├── main.go
└── Dockerfile
```

### 1. `main.go` (Our Sample Go Application)

Create a file named `main.go` inside the `go-app` directory:

```go
package main

import (
    "fmt"
    "log"
    "net/http"
)

func main() {
    http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
        fmt.Fprintf(w, "Hello from my Go Docker app!\n")
    })

    fmt.Println("Server starting on port 8080...")
    log.Fatal(http.ListenAndServe(":8080", nil))
}
```

### 2. Dockerfile (Multi-Stage Build)

Create a file named `Dockerfile` inside the `go-app` directory:

```dockerfile
# Stage 1: Build the Go application
# We name this stage 'builder'
FROM golang:1.22-alpine AS builder

WORKDIR /app

# Copy the Go application source code
COPY main.go .

# Build the Go application
# CGO_ENABLED=0 disables Cgo, making the binary statically linked
# -o app specifies the output filename for the executable
RUN CGO_ENABLED=0 GOOS=linux go build -a -ldflags '-extldflags "-static"' -o app main.go

# Stage 2: Create the final, minimal image
# We use a scratch image (empty base image) for the smallest possible size
FROM scratch

# Copy the compiled application from the 'builder' stage
# Ensure the path matches the output of the build command in the first stage
COPY --from=builder /app/app /app

# Expose the port the application listens on
EXPOSE 8080

# Command to run the application
CMD ["/app"]
```

**Explanation of the Multi-Stage Dockerfile:**

* **`FROM golang:1.22-alpine AS builder`:**
  * This is the first stage, named `builder`.
  * It uses a golang image (which includes the Go compiler and build tools).
  * `AS builder` gives this stage a name, which we can reference later.

* **`WORKDIR /app`:** Sets the working directory inside the builder stage.

* **`COPY main.go .`:** Copies our Go source code into the builder stage.

* **`RUN CGO_ENABLED=0 GOOS=linux go build ...`:**
  * This compiles our Go application.
  * `CGO_ENABLED=0` and `-ldflags '-extldflags "-static"'` ensure the Go binary is statically linked, meaning it doesn't depend on external C libraries that might not be present in the scratch image.
  * `-o app` specifies that the output executable should be named `app`.

* **`FROM scratch`:**
  * This is the second and final stage.
  * `scratch` is the smallest possible base image – it's literally empty. This results in incredibly tiny final images.

* **`COPY --from=builder /app/app /app`:**
  * This is the core of the multi-stage build.
  * `--from=builder` tells Docker to copy files from the stage named `builder`.
  * `/app/app` is the path to our compiled executable within the builder stage.
  * `/app` is the destination path within the current (final) stage.

* **`EXPOSE 8080`:** Informs Docker that the container will listen on port 8080.

* **`CMD ["/app"]`:** Defines the command to run when the container starts, which is our compiled Go application.

### 3. Build the Docker Image

Navigate to the `go-app` directory in your terminal (where `main.go` and `Dockerfile` are located).

Build the Docker image:

```bash
docker build -t my-go-app:latest .
```

This command will execute both stages defined in your Dockerfile. Docker will first build the application in the `builder` stage, then discard that stage's environment and only copy the compiled binary to the `scratch` image.

### 4. Check Image Size

Compare the size of this image with what it would be if you just built it directly on `golang:1.22-alpine` (without multi-stage).

```bash
docker images
```

You will notice `my-go-app:latest` is extremely small (typically a few MBs for the binary itself), whereas a `golang:1.22-alpine` based image with the app would be much larger (hundreds of MBs).

### 5. Run the Container

Run your Go application container:

```bash
docker run -d -p 8080:8080 --name go-web-server my-go-app:latest
```

Verify it's running:

```bash
docker ps
```

Open your web browser and go to http://localhost:8080. You should see "Hello from my Go Docker app!".

### 6. Clean Up

Stop and remove the container:

```bash
docker stop go-web-server
docker rm go-web-server
```

Remove the image:

```bash
docker rmi my-go-app:latest
```

## Benefits Demonstrated

This example showcases the key benefits of multi-stage builds:

1. **Size Reduction:** The final image only contains the compiled Go binary (~5-10MB) instead of the entire Go development environment (~300MB+).
2. **Security:** No build tools or source code in the production image.
3. **Efficiency:** Faster container startup and reduced storage/bandwidth requirements.