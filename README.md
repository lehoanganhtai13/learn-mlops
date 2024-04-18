# Learn MLOps/DevOps
A self-study repo, where there is everything you need for a MLOps system. This could also apply for DevOps system.


## How to use AWS

### AWS CLI

## How to use Docker

### Docker commands
1. Run container

2. Stop running container

3. Build Docker image

### Docker Compose commands
1. Run multi containers (services) of an application
    ```bash
    docker-compose up --build -d
    ```
- `--detach`, `-d`: run containers in the background and leave them running,
- `--no-color`: produce monochrome output.
- `--quiet-pull`: pull without printing progress information.
- `--no-deps`: Don't start linked services.
- `--force-recreate`: recreate containers even if their configuration and image haven't changed.
- `--no-recreate`: if containers already exist, don't recreate them.
- `--no-build`: don't build an image, even if it's missing.
- `--no-start`: don't start the services after creating them.
- `--build`: build images before starting containers.
- `--remove-orphans`: remove containers for services not defined in the Compose file.
- `--timeout TIMEOUT`: use this timeout in seconds for container shutdown when attached or when containers are already running (default is 10).

2. Stop running all the containers
    ```bash
    docker-compose down --rmi all
    ```
- `--rmi all`: remove all images used by any service.
- `--rmi local`: remove only images that don't have a custom tag set by `image` field.
- `--volumes`, `-v`: remove all volumes defined in `volumes` section of the Compose file, but not the named volumes declared in the `docker-compose.yml` file or any other `.yml` files with option `-f`.
- `--remove-orphans`: remove containers for services not defined in the Compose file.
- `--timeout TIMEOUT`, `-t TIMEOUT`: specify a shutdown timeout in seconds (default is 10).

3. Build Docker image seperately
    ```bash
    docker-compose build
    ```
- `--compress`: compress the build context using gzip.
- `--force-rm`: always remove intermediate containers.
- `--no-cache`: do not use cache when building the image to force Docker to run the command again instead of using the cached result.
- `--pull`: always attempt to pull a newer version of the image. When you build a Docker image, it's often based on another image, specified in the `FROM` directive in your Dockerfile. Docker will pull this base image from a Docker registry (like Docker Hub) the first time you build your image.
- `--message`, `-m`: set commit message for the image.
- `SERVICE ...`: optionally specify service(s) to build. If no services are specified, all services defined in the Docker Compose file are built.

