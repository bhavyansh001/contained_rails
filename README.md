# containerd_rails

Commands run:
rails new containerd_rails (to use sqlite3 database)
docker build -t app_image . (built image directly as rails ships with Dockerfile by default)
docker images
docker run -it --user root --rm --entrypoint bash app_image (to get in the container, generate master key if using production as environment using:
        rails credentials:edit)
(Changed ENV to development, else RAILS_MASTER_KEY to be provided using:
        docker run -p 3000:3000 -e RAILS_MASTER_KEY=1234567890 app_image)
CMD[".bin/rails", "server", "-b", "0.0.0.0"] in Dockerfile, to enable binding to all servers

# Developing in Docker using Bind Mounts
docker run -p 3000:3000 -v $(pwd):/rails app_image
This way whatever changes we make will be visible in the app right away without having to rebuild the image

So a single container rails app is up, now let's set up a multi container application
# Multi-container application (branch: multi-container)
Create docker-compose file
Make changes to include pg instead of sqlite3 in Gemfile and Dockerfile, database.yml (host == service_name in docker-compose.yml) and .env file

Commands:
docker-compose exec web bash
-> bin/rails db:create
-> bin/rails g scaffold Post title:string content:text
-> bin/rails db:migrate

docker compose restart web

# Logs:
docker compose logs db
docker compose logs web

# Restarting:
docker-compose down
docker-compose up -d --build (-d: detached)



# Containerd Rails

## Overview
`containerd_rails` is a Rails application designed to run in a Docker container. This guide will walk you through the steps to set up and run the application both as a single container and as a multi-container application using Docker Compose.

## Commands Run

### Initial Setup
1. Create a new Rails application using SQLite3 as the database:
    ```sh
    rails new containerd_rails (app_name)
    ```

2. Build the Docker image:
    ```sh
    docker build -t app_image .
    ```

3. List Docker images to confirm the build:
    ```sh
    docker images
    ```

4. Run the container to access the bash shell (useful for generating a master key if using production environment):
    ```sh
    docker run -it --user root --rm --entrypoint bash app_image
    ```

    To generate a master key:
    ```sh
    rails credentials:edit
    ```

5. For development environment, change `ENV` to development. If using production, provide the `RAILS_MASTER_KEY`:
    ```sh
    docker run -p 3000:3000 -e RAILS_MASTER_KEY=1234567890 app_image
    ```

6. Update the `Dockerfile` to bind the Rails server to all IP addresses:
    ```dockerfile
    CMD [".bin/rails", "server", "-b", "0.0.0.0"]
    ```

## Developing in Docker using Bind Mounts
Run the container with bind mounts to reflect changes in real-time without rebuilding the image:
```sh
docker run -p 3000:3000 -v $(pwd):/rails app_image  (name of the image)
```
## Multi-Container Application

### Setting Up Multi-Container Application (Branch: multi-container)
1. Create a `docker-compose.yml` file.
2. Update the `Gemfile` and `Dockerfile` to use PostgreSQL instead of SQLite3.
3. Modify `database.yml` to set the host as the service name defined in `docker-compose.yml`.
4. Create a `.env` file for environment variables.

### Commands
1. Start the application and access the web container's bash shell:
    ```sh
    docker-compose exec web bash
    ```

2. Create the database and generate a scaffold:
    ```sh
    bin/rails db:create
    bin/rails g scaffold Post title:string content:text
    bin/rails db:migrate
    ```

3. Restart the web service:
    ```sh
    docker-compose restart web
    ```

### Logs
View logs for the database and web services:
    ```sh
    docker-compose logs db
    docker-compose logs web
    ```

### Restarting
To restart the application:

1. Bring down the containers:
    ```sh
    docker-compose down
    ```

2. Bring the containers back up, rebuilding if necessary:
    ```sh
    docker-compose up -d --build
    ```
