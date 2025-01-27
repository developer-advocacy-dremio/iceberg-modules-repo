# Hands-on With Dremio Instructions

Requirements for setting up this environment is having [docker installed](https://www.docker.com).

## Setting Up Environment with Docker Compose

### What is Docker Compose

Docker Compose is a tool that lets us define our environment in a file called `docker-compose.yml`, where we can define multiple containers to run as different 'services' that will get generate on a unique network. 

### Docker Network Trouble Shooting Tips

A dns (domain name service) will be run between the containers on the network, allowing each container to be referenced by their `container_name` like `http://container_name:8080`. If for some reason the DNS isn't resolving (you get errors connecting from one service to another using their container name) then you'll want to use the containers direct ip address on the network docker creates for your environment.

- Find the network by running `docker network ls`

- Then inspect the network with `docker network inspect <network_name>`

This will bring up output that looks like this

```json
[
    {
        "Name": "myproject_default",
        "Id": "6b5d6789abcd",
        "Created": "2024-07-02T12:34:56.789Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.18.0.0/16",
                    "Gateway": "172.18.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Containers": {
            "container_id_1": {
                "Name": "myproject_container_1",
                "EndpointID": "ep1",
                "MacAddress": "02:42:ac:12:00:02",
                "IPv4Address": "172.18.0.2/16",
                "IPv6Address": ""
            },
            "container_id_2": {
                "Name": "myproject_container_2",
                "EndpointID": "ep2",
                "MacAddress": "02:42:ac:12:00:03",
                "IPv4Address": "172.18.0.3/16",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {}
    }
]

```

What you want to look at is this section for the ipv4 address of the particular container:

```json
        "Containers": {
            "container_id_1": {
                "Name": "myproject_container_1",
                "EndpointID": "ep1",
                "MacAddress": "02:42:ac:12:00:02",
                "IPv4Address": "172.18.0.2/16",
                "IPv6Address": ""
            },
            "container_id_2": {
                "Name": "myproject_container_2",
                "EndpointID": "ep2",
                "MacAddress": "02:42:ac:12:00:03",
                "IPv4Address": "172.18.0.3/16",
                "IPv6Address": ""
            }
```

In this case the IP address of the container called `container_id_2` is `172.18.0.3`.

### The Docker Compose File

So in an empty directory somewhere on your computer you will create a `docker-compose.yml` and fill it with the following:

```yaml
services:
  ## Dremio Lakehouse Platform
  dremio:
    platform: linux/x86_64
    image: dremio/dremio-oss:latest
    container_name: dremio
    ports:
      - 9047:9047
      - 31010:31010
      - 32010:32010
      - 45678:45678
    environment:
      - DREMIO_JAVA_SERVER_EXTRA_OPTS=-Dpaths.dist=file:///opt/dremio/data/dist
    networks:
      dremio-iceberg:
  ## Minio Object Storage
  minio:
    image: minio/minio
    container_name: minio
    ports:
      - 9000:9000
      - 9001:9001
    environment:
      - MINIO_ROOT_USER=admin
      - MINIO_ROOT_PASSWORD=password
      - MINIO_DOMAIN=minio
      - MINIO_REGION_NAME=us-east-1
      - MINIO_REGION=us-east-1
    command: server /data --console-address ":9001"
    networks:
        dremio-iceberg:
  ## Nessie Iceberg Catalog
  nessie:
    image: projectnessie/nessie
    container_name: nessie
    ports:
      - "19120:19120"
    networks:
        dremio-iceberg:

networks:
  dremio-iceberg:
```

Then open a terminal to the same directory and you can use the followin commands to work with this file.

- `docker compose up` run every service
- `docker compose down` shuts down all services
- `docker compose up <service_name>` run a particular serivce
- `docker compose down <service_name>` shuts down a particular service
- `docker compose exec <command> <service_name>` runs specified command within a particular service

### additional breakdown of the docker compose command

# Docker Compose Command Flags

#### Common Flags

- `-f, --file FILE`
  - Specify an alternate compose file (default: `docker-compose.yml`).
  - Example: `docker compose -f custom-compose.yml up`

- `-p, --project-name NAME`
  - Specify an alternate project name (default: directory name).
  - Example: `docker compose -p myproject up`

- `--env-file FILE`
  - Specify an alternate environment file.
  - Example: `docker compose --env-file custom.env up`

#### Service Management

- `up`
  - Build, (re)create, start, and attach to containers for a service.
  - Example: `docker compose up`

- `down`
  - Stop and remove containers, networks, images, and volumes.
  - Example: `docker compose down`

- `start`
  - Start existing containers for a service.
  - Example: `docker compose start`

- `stop`
  - Stop running containers without removing them.
  - Example: `docker compose stop`

- `restart`
  - Restart running or stopped containers.
  - Example: `docker compose restart`

- `build`
  - Build or rebuild services.
  - Example: `docker compose build`

#### Logging and Output

- `logs`
  - View output from containers.
  - Example: `docker compose logs`

- `--no-color`
  - Produce monochrome output.
  - Example: `docker compose --no-color up`

#### Scaling and Resources

- `--scale SERVICE=NUM`
  - Scale a service to a specified number of replicas.
  - Example: `docker compose up --scale web=3`

- `--detach, -d`
  - Run containers in the background.
  - Example: `docker compose up -d`

#### Environment and Dependencies

- `--env-file FILE`
  - Specify an alternate environment file.
  - Example: `docker compose --env-file custom.env up`

- `--no-deps`
  - Don’t start linked services.
  - Example: `docker compose up --no-deps SERVICE`

#### Cleanup and Pruning

- `rm`
  - Remove stopped service containers.
  - Example: `docker compose rm`

- `prune`
  - Remove all stopped containers and networks.
  - Example: `docker compose prune`

#### Debugging and Development

- `--build`
  - Rebuild services even if the image already exists.
  - Example: `docker compose up --build`

- `--force-recreate`
  - Recreate containers even if their configuration and image haven't changed.
  - Example: `docker compose up --force-recreate`

- `--no-build`
  - Don’t build an image, even if it’s missing.
  - Example: `docker compose up --no-build`

- `--abort-on-container-exit`
  - Stops all containers if any container was stopped. Useful in conjunction with `--exit-code-from`.
  - Example: `docker compose up --abort-on-container-exit`

- `--exit-code-from SERVICE`
  - Return the exit code of the selected service container.
  - Example: `docker compose up --exit-code-from web`

#### Help and Version

- `--help`
  - Get help on a command.
  - Example: `docker compose up --help`

- `--version`
  - Show the Docker Compose version information.
  - Example: `docker compose --version`

### Setting Up Minio

Once the minio container is running (`docker compose -d up minio`) head over to `http://localhost:9001` and login using the username `admin` and password `password`. Click on "create a bucket" and create two buckets called "warehouse" and "lakehouse".

### Setting up Nessie

Just start the container `docker compose -d up nessie` and your good to go.

[Project Nessie Documentation](https://www.projectnessie.org)

### Setting Up Dremio

Run the command `docker compose up -d dremio` then look at the terminal output for url like this:

Then head over to `localhost:9047` in the browser and setup your dremio admin user.

Once you are on the Dremio UI we are going to add 2 sources

#### Source 1: Nessie Catalog

This will be our Nessie Catalog for Tracking Apache Iceberg Tables, select "add source" choose "nessie".

##### General Settings

- name: nessie
- Nessie endpoint URL: http://nessie:19120/api/v2
- Authentication: none

##### Storage Settings

- aws root path: warehouse
- aws access key: admin
- aws secret key: password
- connection properties:
  - fs.s3a.path.style.access = true
  - fs.s3a.endpoint = minio:9000
  - dremio.s3.compat = true
- encrypt connection: false (unticked)

#### S3/Minio Connection

This will allow us to use the "lakehouse" bucket on minio as a place to upload json, csv, parquet files to work with if we want. Create a new source and select "S3"

##### general settings

- name: lakehouse
- authentication:
  - type: AWS Access Key
  - aws access key: admin
  - aws secret key: password
  - encrypt connection: false

##### advanced options**

- enable compatibility mode: true (ticked)
- root path: /lakehouse
- connection properties:
  - fs.s3a.path.style.access = true
  - fs.s3a.endpoint = minio:9000

Now you can head over to the SQL editor, which is the second item from the top on the sidebar menu on the left.

### Creating Apache Iceberg Tables

#### From Scratch

```sql
-- Creating a namespace for organizing nessie catalog
CREATE FOLDER IF NOT EXISTS nessie.examples AT BRANCH main;

-- Creating a new table
CREATE TABLE IF NOT EXISTS nessie.examples.names (name VARCHAR);

-- Insert Data into the table
INSERT INTO nessie.examples.names VALUES ('Alex'),('Andrew'),('Read');

-- Query the Table
SELECT * FROM nessie.examples.names;
```

#### From Another File

Add another source and select "Dremio Sample Source". Then from the data explorer head over to the "NYC-weather.csv" dataset and select "format" to configure how the CSV file is read as a table. This is an example of having files in an object storage and using Dremio to see them as tables, this makes it really to turn this into an Apache Iceberg table using a CTAS (CREATE TABLE AS) statement.

```sql
-- Turning the CSV file into an Apache Iceberg table using CTAS
CREATE TABLE nessie.examples.weather AS SELECT * FROM Samples."samples.dremio.com"."NYC-weather.csv";

-- Query the Apache Iceberg Version of the Table
SELECT * FROM nessie.examples.weather;
```

### Inserting Data

#### Inserting Data from Another Table

```sql
INSERT INTO nessie.examples.weather SELECT * FROM Samples."samples.dremio.com"."NYC-weather.csv";
```

#### Using COPY INTO to Insert Data From Another File

With files like CSV files, every column will be treated as a text column, but we can avoid this by using the COPY INTO command which copies the data into an existing Apache Iceberg table coercing columns to the destination tables schema.

```sql
-- Create the table to receive CSV data
CREATE TABLE IF NOT EXISTS nessie.examples.weather_copy_into (
 station VARCHAR,
 name VARCHAR,
 "date" TIMESTAMP,
 awnd FLOAT,
 prcp FLOAT,
 snow FLOAT,
 snwd FLOAT,
 tempmax FLOAT,
 tempmin FLOAT
);

-- COPY Data Over From CSV
COPY INTO nessie.examples.weather_copy_into
  FROM '@Samples/samples.dremio.com'
  FILES ('NYC-weather.csv')
  FILE_FORMAT 'csv'
  (TIMESTAMP_FORMAT 'YYYY-MM-DD"T"HH24:MI');

-- Query the Table
SELECT * FROM nessie.examples.weather_copy_into;
```

#### Merge Into

Updating Records and Inserting New Records (Upserts) can be done with Dremio's Merge Into syntax.

```sql
-- Create the target table
CREATE TABLE IF NOT EXISTS nessie.examples.target_table (
  id INTEGER,
  description VARCHAR
);

-- Create the source table
CREATE TABLE IF NOT EXISTS nessie.examples.source_table (
  id INTEGER,
  description_1 VARCHAR,
  description_2 VARCHAR
);

-- Insert initial data into the target table
INSERT INTO nessie.examples.target_table (id, description) VALUES
  (1, 'Original value 1'),
  (2, 'Original value 2');

-- Insert initial data into the source table
INSERT INTO nessie.examples.source_table (id, description_1, description_2) VALUES
  (1, 'Updated value 1', 'Updated value 2'),
  (3, 'New value 1', 'New value 2');

-- View Target table before merging
SELECT * FROM nessie.examples.target_table;

-- Perform the merge operation
MERGE INTO nessie.examples.target_table AS t
USING nessie.examples.source_table AS s
ON t.id = s.id
WHEN MATCHED THEN
  UPDATE SET description = s.description_2
WHEN NOT MATCHED THEN
  INSERT (id, description) VALUES (s.id, s.description_1);

-- View Target table before merging
SELECT * FROM nessie.examples.target_table;
```

### Partitioning

```sql
-- Create the table with initial partitioning on 'name'
CREATE TABLE nessie.examples.partitioned_table (
  id INTEGER,
  name VARCHAR,
  "count" INTEGER,
  ts TIMESTAMP
)
PARTITION BY (name);

-- Insert initial data into the partitioned table
INSERT INTO nessie.examples.partitioned_table VALUES
  (1, 'Alice', NULL, TIMESTAMP '2024-07-02 12:00:00'),
  (2, 'Bob', 5, TIMESTAMP '2024-07-02 12:05:00'),
  (3, 'Catherine', 10, TIMESTAMP '2024-07-02 12:10:00');

-- Query the table before updating partitioning
SELECT * FROM nessie.examples.partitioned_table;

-- Add a new partition field to the table
ALTER TABLE nessie.examples.partitioned_table
  ADD PARTITION FIELD bucket(4, ts);

-- Insert additional data into the updated partitioned table
INSERT INTO nessie.examples.partitioned_table VALUES
  (4, 'David', 15, TIMESTAMP '2024-07-02 12:15:00'),
  (5, 'Eve', 20, TIMESTAMP '2024-07-02 12:20:00');

-- Query the table after updating partitioning
SELECT * FROM nessie.examples.partitioned_table;
```

### Turning off Environment

To shut down all containers run the command `docker compose down` in the folder with the `docker-compose.yml` file.