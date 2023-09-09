# Hello World Nomad

Testing Nomad as Cross Platform Orchestrator

## Getting Started

### Install Nomad on Mac

```bash
brew tap hashicorp/tap
brew install hashicorp/tap/nomad
nomad -v
```

### Create a Nomad Cluster with one node Locally

```bash
sudo nomad agent -dev \
  -bind 0.0.0.0 \
  -network-interface='{{ GetDefaultInterfaces | attr "name" }}'
```

In another terminal

```bash

export NOMAD_ADDR=http://localhost:4646
nomad node status
```

Visit the UI: http://localhost:4646/ui/jobs

### Running Nomad as a Binary (Tauri Sidecar)

1. Download the latest version for your OS [here](https://developer.hashicorp.com/nomad/downloads)

2. Run the following commands
    ```bash
    mv nomad /usr/local/bin
    chmod +x /usr/local/bin/nomad
    ```
3. Verify the installation

    ```bash
    nomad -v
    ```

4. Start the Nomad Agent

    ```bash
    sudo nomad agent -dev 
    ```

### Nomad Example App

Run a database and the web app

```bash
cd ./jobs
nomad job run pytechco-redis.nomad.hcl
nomad job run pytechco-web.nomad.hcl
```

Check the status of the jobs

```bash
nomad node status -verbose \
    $(nomad job allocs pytechco-web | grep -i running | awk '{print $2}') | \
    grep -i ip-address | awk -F "=" '{print $2}' | xargs | \
    awk '{print "http://"$1":5000"}'
```

Submit the setup job to reset the db

```bash
nomad job run pytechco-setup.nomad.hcl
```

Dispatch the setup job by providing a value for budget.

```bash
nomad job dispatch -meta budget="200" pytechco-setup
```

Submit the employee job

```bash
nomad job run pytechco-employee.nomad.hcl
```

Stop the employee job

```bash
nomad job stop -purge pytechco-redis
```

### Check Nomad Stats

You can check resource allocation for each job, node and the entire cluster using Nomad APIs.

```bash
# get nomad system status
curl http://localhost:4646/v1/status/leader
curl http://localhost:4646/v1/status/peers

# get nomad services
curl http://localhost:4646/v1/services

# get nomad process metrics
curl http://localhost:4646/v1/metrics
```