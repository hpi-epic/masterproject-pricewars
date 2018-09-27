<img alt="Price Wars" width="50%" src="/docs/pricewars_logo.svg">

# A Simulation Platform for Dynamic Pricing Competition


This is the meta repository of the Price Wars platform that allows the simulation of a dynamic marketplace similar to online marketplaces like www.amazon.com. Using the simulation, one can test different pricing strategies and assess their performance compared to other pricing strategies.

The simulation is built using a microservice architecture and consists of multiple components representing important players in the simulation. Each component has its own repository and documentation and the links can be found in the following section.

For more information about the platform and publications, see the our chair's [project site](https://hpi.de/en/plattner/projects/price-wars-an-interactive-simulation-platform.html) on Dynamic Pricing under Competition.

On the master branch, this repository contains the docker-setup that allows running the simulation locally. On the [gh-pages](https://github.com/hpi-epic/pricewars/tree/gh-pages) branch one can find the [API specification](https://hpi-epic.github.io/pricewars/) of all components.

## Application Overview

**Repositories**
* Management UI: [https://github.com/hpi-epic/pricewars-mgmt-ui](https://github.com/hpi-epic/pricewars-mgmt-ui)
* Consumer: [https://github.com/hpi-epic/pricewars-consumer](https://github.com/hpi-epic/pricewars-consumer)
* Producer: [https://github.com/hpi-epic/pricewars-producer](https://github.com/hpi-epic/pricewars-producer)
* Marketplace: [https://github.com/hpi-epic/pricewars-marketplace](https://github.com/hpi-epic/pricewars-marketplace)
* Merchant: [https://github.com/hpi-epic/pricewars-merchant](https://github.com/hpi-epic/pricewars-merchant)
* Kafka RESTful API: [https://github.com/hpi-epic/pricewars-kafka-rest](https://github.com/hpi-epic/pricewars-kafka-rest)
* Analytics: [https://github.com/hpi-epic/pricewars-analytics](https://github.com/hpi-epic/pricewars-analytics)

## Architecture

![FMC Architecture Diagram](/docs/modeling/pricewars-architecture.png?raw=true)

## Sequence Diagram

![Sequence Diagram](/docs/modeling/sequence_diagram_flow.png?raw=true)

## Deployment

### Requirements

* [Docker](https://www.docker.com/)
* [Docker Compose](https://docs.docker.com/compose/install/)

### Setup
If you are working on Windows, make sure to set the following git-settings:
```
git config --global core.eol lf
git config --global core.autocrlf input
```
The first setting will ensure that files keep their lf-line endings when checking out a repository.
Otherwise the line endings of all files will be converted to cr+lf under Windows, causing problems with script-execution within the Docker-containers that expect lf-endings.
The second setting ensures that line endings are converted back to what they were on checkout when pushing a change.

*(These settings are global so they will affect all repositories, make sure to change them again if you do not want this behaviour on other repositories!)*

Then go ahead and clone the repository:

```
git clone --recursive git@github.com:hpi-epic/pricewars.git
```

Build docker images and containers with the following command.
Read [this](https://docs.docker.com/install/linux/linux-postinstall/) for running docker as non-root user on Linux.
Bring a fast internet line and some time.
This can take up to 30 minutes to finish.

```
docker-compose up --no-start
```

This command might not be available if you are on an older docker-compose version.
Use `docker-compose create` instead.

Once the containers are created, you can start the Pricewars platform:

```
docker-compose up
```

You can shut down the platform with `CTRL + C` or `docker-compose stop`.

Warning: There might be routing problems if the docker network overlaps with your local network.
If this is the case, change the ip address in `docker-compose.yml` under the `networks` entry.

### Run Pricewars

After starting the Pricewars platform with `docker-compose up`, it can be controlled with the [Management UI](http://localhost)

1. \[Optional] Configure available products in the [Config/Producer section](http://localhost/index.html#/config/producer)
2. Start the [Consumer](http://localhost/index.html#/config/consumer)
3. Merchants are trading products now. The [Dashboard](http://localhost/index.html#/dashboard/overview) shows graphs about sales, profits and more.


#### Cleaning up containers and existing state
Run the following commands to run the platform in a clean state.

```
docker-compose down
docker-compose up
```

#### Updating the Docker setup
First, stop your running containers.

```
docker-compose stop
```

Update repositories.
```
git pull
git submodule update
```

Rebuild all images that have changed:
```
docker-compose build
```

#### Windows w/o Docker Native support
For Windows versions that don't have Docker Native support, like Home etc., another solution is to use an Ubuntu virtual machine with Docker installed. We recommend to use Ubuntu Server as the GUI of other Ubuntu versions might consume to much main memory. The main memory should be at least 8GB, since your Windows consumes about 2GB, Ubuntu Server 0.5GB and the simulation about 4GB. Also, your browser will consume a few hundred MB, too.

#### Help - My Docker Setup is not working as expected!

##### Some containers quit unexpectedly:
First, the analytics container is expected to be stopped after a short moment - it's just used to compile and export the flink jobs. If this is the only container not running, you're fine. In case any other container is not running, analyse the first container (except for analytics) that stopped.
- __Postgres__: Bring the platform in a clean state with `docker-compose down` and run it again.
- __Zookeeper / Kafka__: If you just stopped some older containers: Wait! There is a timeout for Zookeeper to notice that Kafka has been stopped (timestamp-based, so it works even if Zookeeper is not running). Bring the platform in a clean state with `docker-compose rm --stop` and run it again.
- __Others__: Try to read the logs or read on.

##### The command `docker-compose up` is hanging:
- Reset the containers and the network: `docker system prune` (and restart the Docker service).
- Terminate Docker and ensure, that all docker processes are stopped (especially the network service).
- Restart your computer and wait (even though it might be slow) for about five to ten minutes.
- Reset Docker to factory defaults (should be your last attempt, as this requires re-building of all images):
 - macOS: Click on "Preferences" > "Reset" > "Reset to factory defaults"

### Native
For details regarding the deployment of the component, we kindly refer to the deployment section of the microservice specific README.md file. The links can be found above.

## Benchmark Tool

You can run a benchmark on the Pricewars platform with the benchmark tool [benchmark.py](helper_scripts/benchmark.py).
This tool allows to run the platform in a given configuration for a specific time period.
Afterwards, results of this run are written to the output directory.

Example command:
```
python3 helper_scripts/benchmark.py --duration 30 --output <output directory> --merchants <merchant A command> <merchant B command>
```

This starts the whole platform and two merchants to compete against each other for 30 minutes.
As merchant start command you can use for example: `"python3 merchant/merchant.py --strategy Cheapest --port 5000"`
Run `python3 helper_scripts/benchmark.py --help` to see all arguments.

PS: you might need to install several Python libraries (e.g., `matplotlib` via `pip3 install matplotlib`) and Tkinter (e.g., via `sudo apt-get install python3-tk` on Ubuntu).
