# Create a Custom Docker Bridge Network

## How to Create a Custom Docker Bridge Network

Let's create a custom bridge network and demonstrate its benefits with two simple containers.

### Step 1: Check Existing Networks

First, inspect the networks currently available on your Docker daemon:

```bash
docker network ls
```

You will typically see `bridge`, `host`, and `none` as default networks.

### Step 2: Create the Custom Bridge Network

Use the `docker network create` command to create a new custom bridge network. We'll name it `my-app-network`.

```bash
docker network create my-app-network
```

You can optionally specify a subnet and gateway if you need a custom IP range. For example:

```bash
docker network create --driver bridge --subnet 172.18.0.0/16 --gateway 172.18.0.1 my-custom-subnet-network
```

For most cases, Docker's default subnet allocation for custom bridge networks is sufficient.

Verify that your new network has been created:

```bash
docker network ls
```

You should now see `my-app-network` in the list.

### Step 3: Run Containers on the Custom Network

Now, let's run two simple containers and attach them to `my-app-network`.

#### Container 1: A "Database" Service (e.g., Nginx acting as a simple web service)

We'll use an Nginx container and name it `my-db-service` to simulate a backend service.

```bash
docker run -d --name my-db-service --network my-app-network nginx
```

* `-d`: Runs the container in detached mode (background).
* `--name my-db-service`: Assigns a readable name to the container. This name will be used for DNS resolution within the network.
* `--network my-app-network`: Attaches the container to our custom bridge network.
* `nginx`: The Docker image to use.

#### Container 2: A "Web App" Service (e.g., an Alpine container to test connectivity)

We'll use an Alpine container and name it `my-web-app` to simulate a frontend application that needs to talk to `my-db-service`.

```bash
docker run -it --rm --name my-web-app --network my-app-network alpine sh
```

* `-it`: Runs in interactive mode with a TTY.
* `--rm`: Automatically removes the container when it exits.
* `--name my-web-app`: Assigns a readable name.
* `--network my-app-network`: Attaches the container to our custom bridge network.
* `alpine sh`: The Docker image and command to run a shell.

### Step 4: Test DNS Resolution and Connectivity

Once you are inside the `my-web-app` container's shell, you can try to ping or curl the `my-db-service` by its container name:

```bash
# Inside the 'my-web-app' container's shell
ping my-db-service
```

You should see successful ping responses, indicating that the `my-web-app` container can resolve the IP address of `my-db-service` using its name.

Now, try to curl the Nginx web server:

```bash
# Inside the 'my-web-app' container's shell
apk add curl # Install curl if not already present in Alpine
curl http://my-db-service
```

You should see the HTML content of the Nginx welcome page, proving that communication between containers on the same custom network using their names works.

Type `exit` to leave the `my-web-app` container.

### Step 5: Verify Network Details (Optional)

You can inspect your custom network to see which containers are attached and their IP addresses within that network:

```bash
docker network inspect my-app-network
```

Look for the "Containers" section in the JSON output to see `my-db-service` and its details.

### Step 6: Clean Up

Stop and remove the containers:

```bash
docker stop my-db-service
docker rm my-db-service
```

Finally, remove the custom network:

```bash
docker network rm my-app-network
```

Verify that the network is removed:

```bash
docker network ls
```
