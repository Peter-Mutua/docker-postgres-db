# docker-postgres-db
Creating a PostgreSQL Docker container is straightforward. Here’s a step-by-step guide to get a PostgreSQL container up and running:

There are several ways to create a PostgreSQL container

1. Install Docker
Make sure Docker is installed on your system. You can download it from Docker's official website.

2. Pull the PostgreSQL Docker Image
-First, pull the PostgreSQL image from Docker Hub:

$docker pull postgres

-You can specify a version if needed. For example:

$docker pull postgres:15

3.Run a PostgreSQL Container
-To run the PostgreSQL container, use the following command. Replace your_password with the password you want to set for the default postgres user.

$docker run --name my_postgres_container -e POSTGRES_PASSWORD=your_password -d postgres 

This command will:

    Name the container my_postgres_container.
    Set the environment variable POSTGRES_PASSWORD to your_password.
    Run the container in detached mode (-d), so it runs in the background.

4. Customize the Container (Optional)
You can customize it further, such as specifying a database name, user, or port mapping:

$docker run --name my_postgres_container \
  -e POSTGRES_USER=myuser \
  -e POSTGRES_PASSWORD=your_password \
  -e POSTGRES_DB=mydatabase \
  -p 35432:5432 \
  -d postgres

This command:

    Sets POSTGRES_USER to myuser.
    Sets POSTGRES_DB to mydatabase, which will be created upon initialization.
    Maps the container’s port 35432 to the host’s port 5432 (-p 35432:5432), so you can access the database on localhost:35432.

5. Verify the Container is Running

-To check that your PostgreSQL container is running, use:

$docker ps

6. Access PostgreSQL in the Container

-You can connect to the PostgreSQL container from the command line by running:

$docker exec -it my_postgres_container psql -U myuser -d mydatabase

-This command opens a psql prompt where you can run SQL commands against your database.

7. Persistent Data Storage (Optional)

- To ensure data persists even after the container stops, you can mount a volume:

$docker run --name my_postgres_container \
  -e POSTGRES_PASSWORD=your_password \
  -v pgdata:/var/lib/postgresql/data \
  -d postgres

-This creates a Docker volume named pgdata, which will store the database files outside the container.


=To mount a volume for your PostgreSQL Docker container, you have a couple of options:

Option 1: Named Volume (Recommended)

-A named volume is a Docker-managed volume that can be reused and easily referenced by name.

- Here’s how to mount a named volume for PostgreSQL:

 1. Run the container with a named volume:
    
    $docker run --name my_postgres_container \
    -e POSTGRES_PASSWORD=your_password \
    -v pgdata:/var/lib/postgresql/data \
    -d postgres

In this command:

    -v pgdata:/var/lib/postgresql/data mounts the named volume pgdata to /var/lib/postgresql/data in the container, where PostgreSQL stores its data files.

2. Verify the volume is created: 

You can list Docker volumes with:

$docker volume ls

3. Inspect the volume:

To check details about the pgdata volume, use:

$docker volume inspect pgdata

Docker will automatically handle the storage location and lifecycle of named volumes, making them easy to manage and reusable across containers.

Option 2: Bind Mount (Specific Path on Host)

A bind mount maps a specific directory on your host machine to a directory inside the container.

    1. Create a directory on your host:

    Choose a path on your host machine to store the PostgreSQL data. For example:

    $mkdir -p /path/to/your/data

    2. Run the container with a bind mount:

    $docker run --name my_postgres_container \
  -e POSTGRES_PASSWORD=your_password \
  -v /path/to/your/data:/var/lib/postgresql/data \
  -d postgres

  In this command:

    -v /path/to/your/data:/var/lib/postgresql/data mounts the host directory /path/to/your/data to /var/lib/postgresql/data in the container.

   3. Check data persistence:

   With a bind mount, data is directly accessible on your host at /path/to/your/data and will persist even if the container is removed.

   Choosing Between Named Volumes and Bind Mounts

    -Named Volumes are managed by Docker and are ideal if you don’t need to access the data directly on the host.
    -Bind Mounts give you direct access to data on the host but require you to manage the host directory’s location and permissions.


To set up a PostgreSQL Docker container with a custom username and configuration values from a .env file, follow these steps:

Step 1: Create a .env File

- Create a .env file in your project directory with the following variables:

POSTGRES_USER=myuser           # Custom username
POSTGRES_PASSWORD=mypassword    # Password for the custom user
POSTGRES_DB=mydatabase          # Name of the database to create
PGDATA=/var/lib/postgresql/data # Default data directory (or specify a different path if needed)
POSTGRES_PORT=5432              # Port for PostgreSQL

Step 2: Run Docker with Environment Variables from .env

-You can use the --env-file option in Docker to load environment variables from your .env file.

$docker run --name my_postgres_container \
  --env-file .env \
  -p 0.0.0.0:5432:5432 \
  -v pgdata:/var/lib/postgresql/data \
  -d postgres


Explanation of the Command

    --env-file .env: Loads environment variables from the .env file.
    -v pgdata:/var/lib/postgresql/data: Mounts a named volume pgdata to persist data in the container.
    -d postgres: Runs the PostgreSQL image in detached mode.

Step 3: Verify the Container and Environment Variables

To confirm that your container is running with the specified environment variables, you can use:

$docker exec -it my_postgres_container env

This command lists the environment variables in the container, including POSTGRES_USER, POSTGRES_PASSWORD, and POSTGRES_DB.


When you use a named volume like pgdata in Docker, Docker manages the storage location automatically, and it’s stored on your host machine. You can access it by locating the Docker volume directory on your host system.

Here’s how to locate and access the pgdata volume:

1. Locate the Volume Path on Your Host

To find where Docker is storing the pgdata volume:

    Inspect the Volume: Use the following command to see details about the pgdata volume, including its storage path on the host.

    $docker volume inspect pgdata

The output will include a field called "Mountpoint", which shows the full path to where pgdata is stored on your host system. For example:

[
    {
        "CreatedAt": "2024-11-03T10:00:00Z",
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/pgdata/_data",
        "Name": "pgdata",
        "Options": {},
        "Scope": "local"
    }
]

Here, the data is stored at /var/lib/docker/volumes/pgdata/_data.


2. Access the Data

Since Docker manages this directory, accessing it directly requires you to have permissions on the host.

    1. Navigate to the Mountpoint: You can navigate to this path on the host to view the files:

    $cd /var/lib/docker/volumes/pgdata/_data
  
    You should see PostgreSQL data files and folders in this directory, similar to what you would find in a regular PostgreSQL data directory.

    2. Accessing Files: You can read or back up files from this directory, but avoid modifying them directly to prevent corruption in the PostgreSQL instance.

3. Alternative: Bind Mount for Direct Host Access

If you prefer a simpler way to manage and access data, you can use a bind mount instead of a named volume. This lets you specify a path on your host for the PostgreSQL data.

For example:

    1. Create a Directory on the Host:

        $mkdir -p /path/to/your/pgdata

    2. Run the Container with a Bind Mount:

        $docker run --name my_postgres_container \
        --env-file .env \
        -v /path/to/your/pgdata:/var/lib/postgresql/data \
        -d postgres

Now, all PostgreSQL data will be stored in /path/to/your/pgdata on your host, making it easier to access and manage.



You can create a Docker setup that uses a .env file and a custom data directory by writing a Dockerfile and a docker-compose.yml file. Here’s how to set it up:

Since Docker doesn't directly support creating directories based on .env files within a Dockerfile, we’ll use docker-compose, which allows for more flexibility with .env files and volume management.
Step 1: Create a .env File

Add the following configuration in a .env file to set up PostgreSQL variables and the data directory path:

POSTGRES_USER=myuser
POSTGRES_PASSWORD=mypassword
POSTGRES_DB=mydatabase
PGDATA_DIR=./pgdata  # Directory to store PostgreSQL data on the host


Step 2: Create a Dockerfile for PostgreSQL

The Dockerfile will build a custom PostgreSQL image. Here, we’ll start with the official PostgreSQL image and ensure it uses the environment variables.

Create a Dockerfile with the following content:

# Start with the official PostgreSQL image
FROM postgres:latest

# Set environment variables for PostgreSQL
ENV POSTGRES_USER=${POSTGRES_USER}
ENV POSTGRES_PASSWORD=${POSTGRES_PASSWORD}
ENV POSTGRES_DB=${POSTGRES_DB}

# Expose PostgreSQL port
EXPOSE 5432


Step 3: Create a docker-compose.yml File

docker-compose allows you to read environment variables from a .env file, create directories, and mount volumes. This setup will read the .env variables, create the specified data directory, and run the PostgreSQL container.

Here’s how to configure docker-compose.yml

version: '3.8'

services:
  postgres:
    build: .
    container_name: my_postgres_container
    environment:
      - POSTGRES_USER=${POSTGRES_USER}
      - POSTGRES_PASSWORD=${POSTGRES_PASSWORD}
      - POSTGRES_DB=${POSTGRES_DB}
    volumes:
      - ${PGDATA_DIR}:/var/lib/postgresql/data  # Mount the host directory for persistent storage
    ports:
      - "5432:5432"


Step 4: Create the Data Directory

Before running the container, ensure the data directory exists by creating it if it doesn’t already:

$mkdir -p ./pgdata


Step 5: Build and Run the Docker Container

   1. Build the image and start the container using docker-compose:

   $docker-compose up -d

   The -d flag runs it in detached mode, so it runs in the background.

   2. Check that the container is running:

   $docker ps

Build the PostgreSQL image with the specified user, password, and database name.

Mount the pgdata directory on the host as a persistent storage for PostgreSQL data.

Run PostgreSQL on localhost:5432, accessible using the credentials provided in the .env file.
