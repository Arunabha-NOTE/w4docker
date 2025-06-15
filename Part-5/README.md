# Push and pull image to Docker hub and ACR

# Docker Hub: Push and Pull Images

---

## Prerequisites

1.  **Docker Installed:** Ensure Docker Desktop (Windows/macOS) or Docker Engine (Linux) is installed and running on your machine.
2.  **Docker Hub Account:** You need a free Docker Hub account. If you don't have one, sign up at [https://hub.docker.com/](https://hub.docker.com/).
3.  **An Image to Push:** You should have a local Docker image you've created (e.g., using `docker build`) that you want to push. If you don't, you can quickly build one using the Node.js example from the previous part:

    * **Create `my-node-app` directory and files:**
        * `app.js`
        * `package.json`
        * `Dockerfile`
    * **Build the image:**
        ```bash
        cd my-node-app
        docker build -t my-node-app:1.0 .
        ```
    * This will create a local image named `my-node-app` with tag `1.0`.

---

## 1. Login to Docker Hub

Before you can push images, you need to log in to your Docker Hub account from your terminal.

```bash
docker login
```

You will be prompted to enter your Docker Hub username and password.

```
Username: your_dockerhub_username
Password:
Login Succeeded
```

## 2. Tag Your Image (Crucial for Pushing)

Docker Hub requires images to be tagged with your Docker Hub username (or organization name) as a prefix. The format is typically:

```
your_dockerhub_username/repository_name:tag
```

Or, for an organization:

```
organization_name/repository_name:tag
```

Let's assume your Docker Hub username is `your_dockerhub_username`. You need to re-tag your local image `my-node-app:1.0` with this format.

```bash
docker tag my-node-app:1.0 your_dockerhub_username/my-node-app:1.0
```

* `docker tag`: The command to add a new tag to an existing image.
* `my-node-app:1.0`: The source image (local image name and tag).
* `your_dockerhub_username/my-node-app:1.0`: The target image name and tag, including your Docker Hub username.

You can verify the new tag by listing your local images:

```bash
docker images
```

You should now see both `my-node-app:1.0` and `your_dockerhub_username/my-node-app:1.0` pointing to the same IMAGE ID.

### Pro-Tip: Tagging for `latest`

It's common practice to also tag your image with `latest` if it's the most recent stable version:

```bash
docker tag my-node-app:1.0 your_dockerhub_username/my-node-app:latest
```

When you push `latest`, users can pull your image without specifying a version tag (`docker pull your_dockerhub_username/my-node-app`).

## 3. Push Your Image to Docker Hub

Now that your image is correctly tagged, you can push it to your Docker Hub repository.

```bash
docker push your_dockerhub_username/my-node-app:1.0
docker push your_dockerhub_username/my-node-app:latest  # If you also tagged with latest
```

* `docker push`: The command to upload an image to a registry.
* `your_dockerhub_username/my-node-app:1.0`: The full image name including your Docker Hub username and the specific tag you want to push.

You will see output showing the layers being pushed to Docker Hub. Once complete, you can visit `https://hub.docker.com/repositories/your_dockerhub_username` (replace `your_dockerhub_username`) in your web browser to see your newly pushed repository and its tags.

## 4. Pulling Images from Docker Hub

Pulling images from Docker Hub is straightforward. You use the `docker pull` command.

### Pulling Official Images

For official images (like `ubuntu`, `nginx`, `node`), you don't need a username prefix:

```bash
docker pull ubuntu:latest
docker pull nginx:1.25.3
```

### Pulling Your Own or Other Users' Images

To pull images from specific users or organizations, you include the username/organization name:

```bash
# Pull your own image
docker pull your_dockerhub_username/my-node-app:1.0

# Pull an image from another user/organization (e.g., from Docker's own example repo)
docker pull docker/whalesay:latest
```

After pulling, you can verify the image is on your local machine:

```bash
docker images
```

## 5. Running Pulled Images

Once you've pulled an image, you can run it like any other local image:

```bash
# Run your own image
docker run -d -p 8080:3000 --name my-node-container your_dockerhub_username/my-node-app:1.0

# Run the whalesay example
docker run --rm docker/whalesay cowsay "Hello Docker Hub!"
```

## 6. Clean Up

### Remove Local Images

```bash
# Remove local tagged images
docker rmi your_dockerhub_username/my-node-app:1.0
docker rmi your_dockerhub_username/my-node-app:latest
docker rmi my-node-app:1.0

# Remove pulled images
docker rmi ubuntu:latest
docker rmi nginx:1.25.3
```

### Logout from Docker Hub

```bash
docker logout
```

This removes your authentication credentials from the local Docker configuration.

---

# Pushing to ACR
This portion was covered in week 2 assignment so i will be linking that here instead of just writing the steps again.  
Link: [Pushing to ACR](https://github.com/Arunabha-NOTE/AzureInfraAssignment/blob/main/3-create-acr/README.md)