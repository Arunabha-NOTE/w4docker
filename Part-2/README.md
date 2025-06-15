# Docker Installation and Basic Container Operations

## 1. Docker Installation

Docker offers different installation methods depending on your operating system.

### For Windows

Docker Desktop is the easiest way to get Docker up and running on your local machine. It includes Docker Engine, Docker CLI client, Docker Compose, Kubernetes, and Credential Helper.

1.  **Download Docker Desktop:**
    * **For Windows:** Visit [Docker Desktop for Windows](https://docs.docker.com/desktop/install/windows-install/)  

2.  **Run the Installer:**
    * Follow the on-screen instructions. For Windows, ensure "Use WSL 2 instead of Hyper-V (recommended)" is checked during installation if you have WSL 2 enabled.  

3.  **Start Docker Desktop:**
    * Once installed, launch Docker Desktop from your applications.
    * The Docker icon will appear in your system tray indicating that Docker Engine is running.

4.  **Verify Installation:**
    * Open your terminal or command prompt and run:
        ```bash
        docker --version
        docker compose version
        ```
    * You should see the installed Docker and Docker Compose versions.

### 2.1. Running a Simple Container

Run an Nginx web server container and map port 8080 on your host to port 80 inside the container.

```bash
docker run -d -p 8080:80 --name my-nginx-app nginx
```

* `-d`: Runs the container in "detached" mode (in the background).
* `-p 8080:80`: Maps host port 8080 to container port 80.
* `--name my-nginx-app`: Assigns a readable name to your container.
* `nginx`: The name of the Docker image to use.

Verify it's running:

```bash
docker ps
```

Open your browser and navigate to http://localhost:8080. You should see the Nginx welcome page.

### 2.2. Interacting with a Container

Get an interactive shell inside a running container (e.g., your my-nginx-app container).

```bash
docker exec -it my-nginx-app bash
```

Now you are inside the container's shell. You can explore its file system, run commands (e.g., `ls -l /etc/nginx/`), and then exit to return to your host terminal.

### 2.3. Stopping and Removing Containers

Stop the Nginx container:

```bash
docker stop my-nginx-app
```

Remove the Nginx container:

```bash
docker rm my-nginx-app
```

Verify it's gone:

```bash
docker ps -a
```

You should no longer see my-nginx-app in the list.

### 2.4. Cleaning Up Images

Remove the nginx image that was pulled:

```bash
docker rmi nginx
```

Verify the image is removed:

```bash
docker images
```

## 3. Building an Image from a Dockerfile

A Dockerfile is a text file that contains all the commands a user could call on the command line to assemble an image. It's the blueprint for your Docker image.

Let's create a simple web application that displays "Hello from my Docker app!" and package it into a Docker image.

### Step 3.1: Create Project Directory

Create a new directory for your project:

```bash
mkdir my-docker-app
cd my-docker-app
```

### Step 3.2: Create the Web Application File

Inside my-docker-app, create a file named `app.py` with the following content:

```python
# app.py
from flask import Flask
app = Flask(__name__)

@app.route('/')
def hello():
    return 'Hello from my Docker app!'

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

### Step 3.3: Create requirements.txt

Also in my-docker-app, create a file named `requirements.txt` with the following content:

```
Flask==2.3.3
```

### Step 3.4: Create the Dockerfile

Inside my-docker-app, create a file named `Dockerfile` (no file extension) with the following content:

```dockerfile
# Use an official Python runtime as a parent image
FROM python:3.9-slim-buster

# Set the working directory in the container to /app
WORKDIR /app

# Copy the current directory contents into the container at /app
COPY . /app

# Install any needed packages specified in requirements.txt
RUN pip install --no-cache-dir -r requirements.txt

# Make port 5000 available to the world outside this container
EXPOSE 5000

# Run app.py when the container launches
CMD ["python", "app.py"]
```

### Step 3.5: Build the Docker Image

Navigate to your my-docker-app directory in your terminal (where your Dockerfile, app.py, and requirements.txt are located).

Now, build the image using the docker build command:

```bash
docker build -t my-flask-app:1.0 .
```

You will see output indicating each step of the build process. Once complete, you can verify your new image:

```bash
docker images
```

### Step 3.6: Run Your Custom Image

Now, run a container from your newly built image:

```bash
docker run -d -p 5000:5000 --name my-custom-web-app my-flask-app:1.0
```

Verify the container is running:

```bash
docker ps
```

Open your web browser and navigate to http://localhost:5000. You should see "Hello from my Docker app!".

### Step 3.7: Clean Up

Stop and remove your custom application container:

```bash
docker stop my-custom-web-app
docker rm my-custom-web-app
```

Remove your custom image:

```bash
docker rmi my-flask-app:1.0
```