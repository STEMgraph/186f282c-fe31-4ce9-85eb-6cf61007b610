<!---
{
  "id": "186f282c-fe31-4ce9-85eb-6cf61007b610",
  "depends_on": ["d1bee1c7-d88a-4f00-a44e-3e402f6ee826"],
  "author": "Stephan Bökelmann",
  "first_used": "2025-05-27",
  "keywords": ["InfluxDB", "Docker", "Container", "Persistent Storage", "Time Series", "Flux"]
}
--->

# Using InfluxDB v2 in a Docker Container

> In this exercise you will learn how to deploy and interact with an InfluxDB v2 database using Docker containers. Furthermore, we will explore how data persistence affects containerized databases and how to prevent data loss using volume mounts.

### Introduction

InfluxDB is a high-performance time series database designed for storing and querying large volumes of timestamped data, such as metrics and events. With the release of InfluxDB v2, significant changes have been introduced, including a unified API, a new query language called Flux, and an integrated UI for data exploration.

Deploying InfluxDB within Docker containers offers a streamlined approach to setting up and managing the database environment. However, it's crucial to understand how data persistence works in this context. By default, Docker containers are ephemeral, meaning that any data stored inside the container is lost when the container is removed. To ensure data durability, especially for databases, it's essential to configure persistent storage using Docker volumes.

In this exercise, we will guide you through the process of running InfluxDB v2 in a Docker container, initially without persistent storage to observe the effects, and then with a Docker volume to maintain data across container restarts and removals. You'll learn how to interact with InfluxDB using its CLI and UI, create buckets (analogous to databases), write and query data using Flux, and understand the importance of data persistence in containerized environments.

### Further Readings and Other Sources

* [InfluxDB Docker Hub Repository](https://hub.docker.com/_/influxdb)
* [InfluxDB v2 Documentation](https://docs.influxdata.com/influxdb/v2/)
* [Docker Volumes Documentation](https://docs.docker.com/storage/volumes/)
* [Flux Language Reference](https://docs.influxdata.com/flux/v0.x/)
* [InfluxDB v2 Tutorial](https://www.youtube.com/watch?v=BcckJKke_dE)

### Tasks

1. **Pull the InfluxDB v2 Image:**

   ```bash
   docker pull influxdb:2
   docker images | grep influxdb
   ```

2. **Run an InfluxDB Container:**

   ```bash
   docker run --name influxdb-temp -p 8086:8086 -d influxdb:2
   docker ps -a
   ```

3. **Access the InfluxDB UI and Perform Initial Setup:**
   Visit `http://localhost:8086` and set up with:

   * Username: `admin`
   * Password: `admin123`
   * Organization: `example-org`
   * Bucket: `example-bucket`

4. **Write Data to the Bucket:**

   ```bash
   docker exec -it influxdb-temp influx write \
     --bucket example-bucket \
     --org example-org \
     --precision s \
     --token <your-token> \
     --body 'temperature,location=office value=23.5 1625865600'
   ```

5. **Query the Data Using Flux:**

   ```flux
   from(bucket: "example-bucket")
     |> range(start: -1h)
     |> filter(fn: (r) => r._measurement == "temperature")
   ```

6. **Stop and Remove the Container:**

   ```bash
   docker stop influxdb-temp
   docker rm influxdb-temp
   ```

7. **Run a New Container Without Persistent Storage:**

   ```bash
   docker run --name influxdb-temp -p 8086:8086 -d influxdb:2
   ```

8. **Create a Docker Volume for Persistence:**

   ```bash
   docker volume create influxdb2-data
   docker run --name influxdb-persistent -p 8086:8086 \
     -v influxdb2-data:/var/lib/influxdb2 \
     -v influxdb2-config:/etc/influxdb2 \
     -e DOCKER_INFLUXDB_INIT_MODE=setup \
     -e DOCKER_INFLUXDB_INIT_USERNAME=admin \
     -e DOCKER_INFLUXDB_INIT_PASSWORD=admin123 \
     -e DOCKER_INFLUXDB_INIT_ORG=example-org \
     -e DOCKER_INFLUXDB_INIT_BUCKET=example-bucket \
     -d influxdb:2
   ```

9. **Verify Data Persistence:**

   ```bash
   docker stop influxdb-persistent
   docker rm influxdb-persistent

   docker run --name influxdb-persistent -p 8086:8086 \
     -v influxdb2-data:/var/lib/influxdb2 \
     -v influxdb2-config:/etc/influxdb2 \
     -d influxdb:2
   ```

### Questions

1. What happens to the InfluxDB data when the container is deleted without persistent storage?
2. Why is using a volume critical for database containers?
3. How do you list all Docker volumes on your system?
4. Can you explain the command `-v influxdb2-data:/var/lib/influxdb2` in detail?
5. What alternative methods besides named volumes could you use to persist data in Docker?

### Advice

Understanding how to manage data persistence in containerized environments is crucial for maintaining reliable and consistent applications. InfluxDB v2, with its time series capabilities, is a powerful tool for monitoring and analytics. However, without proper volume configuration, all your data can be lost upon container removal. By practicing the setup with and without persistent storage, you gain insight into Docker's behavior and the importance of volumes. This knowledge is not only applicable to InfluxDB but extends to other stateful services you may deploy in the future. Remember to explore additional features of InfluxDB v2, such as its integrated UI, Flux language, and API capabilities, to fully leverage its potential in your projects.
