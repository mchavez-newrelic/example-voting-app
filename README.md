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
<details>
 <summary>Account Setup</summary>
 
- You'll need a New Relic account. The good news is that you can create a free account [here](https://newrelic.com/signup) (no credit card required).
</details>
<details>
 <summary>Python Agent Installation</summary>
 
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
* Next, connect your infrastructure by running the given Docker command as shown below. <img width="971" alt="Screenshot 2023-06-06 at 3 53 40 PM" src="https://github.com/mchavez-newrelic/example-voting-app/assets/104166698/01259262-1d93-4238-9f45-5655d2dfd7d5">
* Run your application with `docker compose up` in the project directory
* Finally, test the connection to the Python agent and your infrastructure. You should see results similar to the screenshot below. It is ok for the `On-host logs` connection to fail. ![image](https://github.com/mchavez-newrelic/example-voting-app/assets/132291725/053d87b0-81e4-4dd7-8421-1ec2443ef65c)
</details>
 
<details>
 <summary>Troubleshooting Python Agent Installation</summary>
 
* If the connection to the Python agent fails in the last step. 
  * First tear down your Docker containers with `docker compose down`
  * Try running the following command to forcefully rebuild your images: `docker compose build --no-cache`
  * Then try running `docker compose up` again to start your containers
* If the connection to the Infrastructure agent fails in the last step.
  * Try running the Linux install command instead of the Docker command as shown below. <img width="962" alt="Screenshot 2023-06-06 at 4 05 09 PM" src="https://github.com/mchavez-newrelic/example-voting-app/assets/104166698/e907fb62-2888-4bae-bef5-610b38012403">
</details>

## Building a Dashboard
<details>
 <summary>Instructions</summary>
 
 * In your New Relic One dashboard on the left, click on `Dashboards`, then on the top right, click on `Create a dashboard`
 <img width="1512" alt="Screenshot 2023-06-07 at 3 34 54 PM" src="https://github.com/mchavez-newrelic/example-voting-app/assets/104166698/d4551df2-6b7d-40d6-b912-245285457dae">
 
 * Select `Create a new dashboard` and on the next page, enter a name before proceeding.
 <img width="406" alt="Screenshot 2023-06-07 at 3 35 20 PM" src="https://github.com/mchavez-newrelic/example-voting-app/assets/104166698/c993b574-8c51-4bf3-b465-a566774a210a"> <img width="404" alt="Screenshot 2023-06-07 at 3 36 12 PM" src="https://github.com/mchavez-newrelic/example-voting-app/assets/104166698/1c31b155-1f3d-41f2-b845-c2b1653045de">

 * Click on any panel to `Add a new chart` and in the pop-up on the right, select `Add a chart`
 <img width="434" alt="Screenshot 2023-06-07 at 2 46 41 PM" src="https://github.com/mchavez-newrelic/example-voting-app/assets/104166698/5fb9e67b-eb62-4a16-90e0-1bbd65effbc1"> <img width="396" alt="image" src="https://github.com/mchavez-newrelic/example-voting-app/assets/104166698/02ddff53-02fd-4390-80bd-1948287eca42">
 
 * Enter the follow query `SELECT count(*) FROM votes FACET appName TIMESERIES SINCE 5 hours ago` and hit the `Run` button. You should be able to see a chart of the results being sent from the application. You can also customize how the chart looks, for example, we can change the "Chart Type"
 <img width="1360" alt="Screenshot 2023-06-07 at 2 47 51 PM" src="https://github.com/mchavez-newrelic/example-voting-app/assets/104166698/c78e8c35-fe48-4c7c-80d1-55d6e286102a">
 
 * Changing the "Chart Type" to Stacked Bar will look like this:
<img width="1361" alt="Screenshot 2023-06-07 at 2 47 33 PM" src="https://github.com/mchavez-newrelic/example-voting-app/assets/104166698/96b8f6a1-298e-47e1-837c-57127eecd212">
</details>

### .NET Agent Installation
<details>
  <summary style="background-color: #1DE783; color: black; padding: 10px;">Steps for .NET Agent</summary>
 1. You can begin installing the .NET agent by first clicking the <b>Add Data</b> tab on the left hand navigation pane, as shown below. <br/><img width="1490" alt="Screenshot 2023-06-05 at 2 36 30 PM" src="https://github.com/mchavez-newrelic/relicstaurants/assets/132291725/5fccb01f-e9c4-4877-b977-7df2ff5c2553">
 2. Search for the .NET agent in the <b>Search for any technology</b> search bar and click the Python agent under the <b>Application monitoring</b> section as shown below. <br/><img width="824" alt="image" src="https://github.com/mchavez-newrelic/example-voting-app/assets/132291725/6f3085b5-0778-4e4c-a5e9-55d61ae48afb"><br/>
3. Next, give your application a name, preferably different from the name given to your Python Agent. For example, you can name the .NET application <code>example-voting-app-worker</code> in your New Relic account.
 4. We will be following the steps linked <a href="https://docs.newrelic.com/install/dotnet/?deployment=linux&docker=yesDocker">here</a> to install and enable the .NET agent inside our .NET Docker container.
<ul>
  <li>Replace the code in your <code>/worker/Dockerfile</code> file for your .NET worker to be as shown <a href="https://github.com/mchavez-newrelic/example-voting-app/blob/instrumented-version/worker/Dockerfile">here</a>.</li>
  <li>Make sure to replace <code>YOUR_LICENSE_KEY</code> and <code>YOUR_APP_NAME</code> with your New Relic license key and .NET application name respectively inside the <code>ENV</code> command at the bottom of the Dockerfile. If you would like to know where to find your license key, you can follow instructions <a href="https://docs.newrelic.com/docs/apis/intro-apis/new-relic-api-keys/">here</a>.</li>
 </ul>
</details>

 ### .NET Worker Custom Instrumentation
 <details>
  <summary style="background-color: #1DE783; color: black; padding: 10px;">Adding Custom Instrumentation</summary>
  1. If you have installed the .NET agent inside the .NET worker Dockerfile, we can begin adding custom instrumentation to monitor the .NET worker's transactions. 
 <ul>
  <li>Let's first make sure we install the <code>NewRelic.Agent.Api</code> package in our project's PackageReference.</li>
  <li>Replace your <code>/worker/Worker.csproj</code> file with the code <a href="https://github.com/mchavez-newrelic/example-voting-app/blob/418fd6dcbd60642ec2ab30932827b934711cec9f/worker/Worker.csproj#LL12C6-L12C6">here</a> so we can install the <code>NewRelic.Agent.Api</code> package.</li>
 </ul>
 2. Let's begin with a simple task of tracking the <code>UpdateVote</code> transaction inside the <code>/worker/Program.cs</code> file for the .NET worker.
 </details>
 

 


## Notes

The voting application only accepts one vote per client browser. It does not register additional votes if a vote has already been submitted from a client.

This isn't an example of a properly architected perfectly designed distributed app... it's just a simple
example of the various types of pieces and languages you might see (queues, persistent data, etc), and how to
deal with them in Docker at a basic level.
