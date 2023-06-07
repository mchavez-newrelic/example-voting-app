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
<details>
  <summary markdown='span' style="background-color: #1DE783; color: black; padding: 10px;">Steps for Python Agent Installation</summary>
 1. You can begin installing the Python agent by first clicking the <b>Add Data</b> tab on the left hand navigation pane, as shown below. <br/><img width="1490" alt="Screenshot 2023-06-05 at 2 36 30 PM" src="https://github.com/mchavez-newrelic/relicstaurants/assets/132291725/5fccb01f-e9c4-4877-b977-7df2ff5c2553">
 2. Search for the Python agent in the <b>Search for any technology</b> search bar and click the Python agent under the <b>Application monitoring</b> section as shown below. <br/><img width="824" alt="image" src="https://github.com/mchavez-newrelic/example-voting-app/assets/132291725/6f3085b5-0778-4e4c-a5e9-55d61ae48afb"><br/>
3. Next, give your application a name.<br/>
4. Install the New Relic agent into the voting app Docker container by following the below steps:
<ul>
 <li>Add the <code>newrelic</code> Python module as a dependency in the <code>/vote/requirements.txt</code> file</li>
 <li>Copy the <code>newrelic.ini</code> file as shown below into the <code>/vote</code> directory of the project folder. <br/><img width="1245" alt="image" src="https://github.com/mchavez-newrelic/example-voting-app/assets/132291725/8f2c5ad0-348b-466e-9d9c-a5409b5c08b2"></li>
 <li>Add the <code>NEW_RELIC_CONFIG_FILE</code> as an environment variable in the <code>docker-compose.yml</code> file to point to the <code>newrelic.ini</code> file.</li>
 <li>Add the <code>newrelic-admin run-program</code> commands in front of the existing <code>python app.py</code> command for the vote Docker service.</li>
 <li>Your <code>vote</code> service in your <code>docker-compose.yml</code> file should look like the code <a href="https://github.com/mchavez-newrelic/example-voting-app/blob/418fd6dcbd60642ec2ab30932827b934711cec9f/docker-compose.yml#LL6C4-L6C4">here</a>.</li>
 </ul>
5. Next, connect your infrastructure by running the given Docker command as shown below. <img width="1259" alt="image" src="https://github.com/mchavez-newrelic/example-voting-app/assets/132291725/a769e827-056e-4518-b74c-13907e4a9d0b">
6. Run your application with <code>docker compose up</code> in the project directory
7. Finally, test the connection to the Python agent and your infrastructure. You should see results similar to the screenshot below. It is ok for the <b>On-host logs</b> connection to fail. <img width="1259" alt="image" src="https://github.com/mchavez-newrelic/example-voting-app/assets/132291725/b2a9f949-a276-4f2a-bfdc-0cf2fdc10c93">
</details>

### Troubleshooting Python Agent Installation
<details>
  <summary style="background-color: #1DE783; color: black; padding: 10px;">Troubleshooting Python Agent Installation</summary>
1. If the connection to the Python agent fails in the last step. 
 <ul>
  <li>First tear down your Docker containers with <code>docker compose down</code></li>
  <li>Try running the following command to forcefully rebuild your images: <code>docker compose build --no-cache</code></li>
  </li>Then try running <code>docker compose up</code> again to start your containers</li>
 </ul>
2. If the connection to the Infrastructure agent fails in the last step, try running the Linux install command instead of the Docker command as shown below.<br/><img width="1255" alt="image" src="https://github.com/mchavez-newrelic/example-voting-app/assets/132291725/438cb11f-fe7f-45be-b192-93fd7c512839">
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
