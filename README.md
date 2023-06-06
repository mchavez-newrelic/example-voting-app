# Example Voting App

A simple distributed application running across multiple Docker containers.

## Getting started

Download [Docker Desktop](https://www.docker.com/products/docker-desktop) for Mac or Windows. [Docker Compose](https://docs.docker.com/compose) will be automatically installed. On Linux, make sure you have the latest version of [Compose](https://docs.docker.com/compose/install/).

This solution uses Python, Node.js, .NET, with Redis for messaging and Postgres for storage.

Run in this directory to build and run the app:

```shell
docker compose up
```

The `vote` app will be running at [http://localhost:5000](http://localhost:5000), and the `results` will be at [http://localhost:5001](http://localhost:5001).

Alternately, if you want to run it on a [Docker Swarm](https://docs.docker.com/engine/swarm/), first make sure you have a swarm. If you don't, run:

```shell
docker swarm init
```

Once you have your swarm, in this directory run:

```shell
docker stack deploy --compose-file docker-stack.yml vote
```

## Run the app in Kubernetes

The folder k8s-specifications contains the YAML specifications of the Voting App's services.

Run the following command to create the deployments and services. Note it will create these resources in your current namespace (`default` if you haven't changed it.)

```shell
kubectl create -f k8s-specifications/
```

The `vote` web app is then available on port 31000 on each host of the cluster, the `result` web app is available on port 31001.

To remove them, run:

```shell
kubectl delete -f k8s-specifications/
```

## Architecture

![Architecture diagram](architecture.excalidraw.png)

* A front-end web app in [Python](/vote) which lets you vote between two options
* A [Redis](https://hub.docker.com/_/redis/) which collects new votes
* A [.NET](/worker/) worker which consumes votes and stores them in…
* A [Postgres](https://hub.docker.com/_/postgres/) database backed by a Docker volume
* A [Node.js](/result) web app which shows the results of the voting in real time

## Instrumentation
### Account Setup
- You'll need a New Relic account. The good news is that you can create a free account [here](https://newrelic.com/signup) (no credit card required).
### Python Agent Installation
* Once you've created an account, you can begin installing the agent by first clicking the `Add Data` tab on the left hand navigation pane, as shown below. <img width="1490" alt="Screenshot 2023-06-05 at 2 36 30 PM" src="https://github.com/mchavez-newrelic/relicstaurants/assets/132291725/5fccb01f-e9c4-4877-b977-7df2ff5c2553">
* Search for the Python agent in the `Search for any technology` search bar and click the Python agent under the `Application monitoring` section as shown below. <img width="824" alt="image" src="https://github.com/mchavez-newrelic/example-voting-app/assets/132291725/6f3085b5-0778-4e4c-a5e9-55d61ae48afb">
* Next, give your application a name
* Install the New Relic agent into the voting app Docker container by following the below steps
  * Add the `newrelic` Python module as a dependency in the `/vote/requirements.txt` file
  * Copy the `newrelic.ini` file as shown below into the `/vote` directory of the project folder. <img width="1245" alt="image" src="https://github.com/mchavez-newrelic/example-voting-app/assets/132291725/8f2c5ad0-348b-466e-9d9c-a5409b5c08b2">
  * Add the `NEW_RELIC_CONFIG_FILE` as an environment variable in the `docker-compose.yml` file to point to the `newrelic.ini` file. 
  * Add the `newrelic-admin run-program` commands in front of the existing `python app.py` command for the vote Docker service.
  * Your `vote` service in your `docker-compose.yml` file should look like the following:
```
services:
  vote:
    build: ./vote
    # use python rather than gunicorn for local dev
    command: newrelic-admin run-program python app.py
    environment:
      - NEW_RELIC_CONFIG_FILE=/app/newrelic.ini
    depends_on:
      redis:
        condition: service_healthy
    healthcheck: 
      test: ["CMD", "curl", "-f", "http://localhost"]
      interval: 15s
      timeout: 5s
      retries: 3
      start_period: 10s
    volumes:
     - ./vote:/app
    ports:
      - "5000:80"
    networks:
      - front-tier
      - back-tier
```
* Next, connect your infrastructure by running the given Docker command as shown below. <img width="1259" alt="image" src="https://github.com/mchavez-newrelic/example-voting-app/assets/132291725/a769e827-056e-4518-b74c-13907e4a9d0b">
* Run your application with `docker compose up` in the project directory
* Finally, test the connection to the Python agent and your infrastructure. You should see results similar to the screenshot below. It is ok for the `On-host logs` connection to fail. ![image](https://github.com/mchavez-newrelic/example-voting-app/assets/132291725/053d87b0-81e4-4dd7-8421-1ec2443ef65c)

### Troubleshooting Python Agent Installation
* If the connection to the Python agent fails in the last step. 
  * First tear down your Docker containers with `docker compose down`
  * Try running the following command to forcefully rebuild your images: `docker compose build --no-cache`
  * Then try running `docker compose up` again to start your containers
* If the connection to the Infrastructure agent fails in the last step.
  * Try running the Linux install command instead of the Docker command as shown below. <img width="1255" alt="image" src="https://github.com/mchavez-newrelic/example-voting-app/assets/132291725/438cb11f-fe7f-45be-b192-93fd7c512839">





## Notes

The voting application only accepts one vote per client browser. It does not register additional votes if a vote has already been submitted from a client.

This isn't an example of a properly architected perfectly designed distributed app... it's just a simple
example of the various types of pieces and languages you might see (queues, persistent data, etc), and how to
deal with them in Docker at a basic level.
