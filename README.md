# Week 6: FastAPI in Docker project

## The API

The API this week is a single-file version of the Courses API developed in
the lab in week 3.  Like the week 3 version, it requires FastAPI, Pydantic,
SQLAlchemy, and Uvicorn.

## Docker commands

Each of these should be run from the `week6-fastapi-docker` directory (i.e., the
same directory as the one containing this `README.md`).

Start off by doing the following:

    DOCKERHUB_USERNAME=<dockerhub-username>
    version=1
    docker login --username "$DOCKERHUB_USERNAME"

### Build

    docker build -t "week6-courses-api:$version" .

### Tag

    docker tag "week6-courses-api:$version" "$DOCKERHUB_USERNAME/week6-courses-api:$version"

### Publish

    docker push "$DOCKERHUB_USERNAME/week6-courses-api:$version"

### Pull

    docker pull "$DOCKERHUB_USERNAME/week6-courses-api:$version"

### Run

    docker run -d --name courses-api -p 80:8000 --env-file .env \
        --mount type=volume,source=week6-courses-data,target=/data \
        "$DOCKERHUB_USERNAME/week6-courses-api:$version"

## My Docker Hub `week6-fastapi-docker` image

    se93jmh0/week6-courses-api@sha256:e24422c00bb2fc1010397fff1acbb8a20f3b4c91fc1e20fc108837363f821f14

## Docker volume

The command for running, above, uses a volume to ensure that Course data is
retained across containers.  In the example the name of the volume is
`week6-courses-data` and it is mounted at `/data`.  The `.env` file is the
magic that ensures that the SQLite database is then written to the named volume
rather than the container's filesystem:

    2026-Nov-18 19:05:01 UTC student@lab-5
    0 302 ~/week6-fastapi-docker
    $ cat .env
    DATABASE_URL=sqlite:////data/courses.db


    2026-Nov-18 19:14:29 UTC student@lab-5
    0 303 ~/week6-fastapi-docker
    $ 

The volume can be inspected like so:

    2026-Nov-18 19:16:48 UTC student@lab-5
    0 307 ~/week6-fastapi-docker
    $ docker volume inspect week6-courses-data
    [
        {
            "CreatedAt": "2026-11-12T20:27:33Z",
            "Driver": "local",
            "Labels": null,
            "Mountpoint": "/var/lib/docker/volumes/week6-courses-data/_data",
            "Name": "week6-courses-data",
            "Options": null,
            "Scope": "local"
        }
    ]

    2026-Nov-18 19:16:57 UTC student@lab-5
    0 307 ~/week6-fastapi-docker
    $ 

And its possible to show its being used thusly:

    2026-Nov-18 19:18:38 UTC student@lab-5
    0 308 ~/week6-fastapi-docker
    $ docker inspect --format '{{.Id}} {{.Config.Image}}{{range .Mounts}} {{.Name}} → {{.Destination}}{{end}}' courses-api
    a524a159d7559441f418a63e7a417f45aa4ecc4f592ca3282d0600cadb1e2955 se93jmh0/week6-courses-api:1 week6-courses-data → /data

    2026-Nov-18 19:18:47 UTC student@lab-5
    0 308 ~/week6-fastapi-docker
    $ 

## Testing

### API home

    2026-Nov-18 19:20:53 UTC student@lab-5
    0 309 ~/week6-fastapi-docker
    $ curl -i http://127.0.0.1/
    HTTP/1.1 200 OK
    date: Wed, 18 Nov 2026 19:21:02 GMT
    server: uvicorn
    content-length: 35
    content-type: application/json

    {"message":"Courses API version 1"}
    2026-Nov-18 19:21:02 UTC student@lab-5
    0 309 ~/week6-fastapi-docker
    $ 

### Creating a Course

    2026-Nov-18 19:21:39 UTC student@lab-5
    0 309 ~/week6-fastapi-docker
    $     curl -i http://127.0.0.1/courses -H "Content-Type: application/json" \
            -d '{ "code": "ACC29", "title": "Advanced Cloud Computing [2029 edition]", "lecturer": "Donald Knuth" }'
    HTTP/1.1 201 Created
    date: Wed, 18 Nov 2026 19:24:38 GMT
    server: uvicorn
    content-length: 99
    content-type: application/json

    {"code":"ACC29","title":"Advanced Cloud Computing [2029 edition]","lecturer":"Donald Knuth","id":4}
    2026-Nov-18 19:24:38 UTC student@lab-5
    0 309 ~/week6-fastapi-docker
    $ 

### Querying a Course

    curl -i http://127.0.0.1/courses/<course-id>

E.g.:

    2026-Nov-18 19:26:03 UTC student@lab-5
    0 310 ~/week6-fastapi-docker
    $ curl -i http://127.0.0.1/courses/4
    HTTP/1.1 200 OK
    date: Wed, 18 Nov 2026 19:26:03 GMT
    server: uvicorn
    content-length: 99
    content-type: application/json

    {"code":"ACC29","title":"Advanced Cloud Computing [2029 edition]","lecturer":"Donald Knuth","id":4}
    2026-Nov-18 19:26:04 UTC student@lab-5
    0 310 ~/week6-fastapi-docker
    $

## Replacing the container

    docker ps && \
        docker stop courses-api && \
        docker rm courses-api && \
        docker run -d --name courses-api -p 80:8000 --env-file .env \
            --mount type=volume,source=week6-courses-data,target=/data \
            "$DOCKERHUB_USERNAME/week6-courses-api:$version" && \
        docker ps

E.g.:

    2026-Nov-18 19:31:38 UTC student@lab-5
    0 313 ~/week6-fastapi-docker
    $ docker ps && \
        docker stop courses-api && \
        docker rm courses-api && \
        docker run -d --name courses-api -p 80:8000 --env-file .env \
            --mount type=volume,source=week6-courses-data,target=/data \
            "$DOCKERHUB_USERNAME/week6-courses-api:$version" && \
        docker ps
    CONTAINER ID   IMAGE                          COMMAND                  CREATED          STATUS          PORTS                                     NAMES
    23f997b9ff8e   se93jmh0/week6-courses-api:1   "uvicorn main:app --…"   27 seconds ago   Up 27 seconds   0.0.0.0:80->8000/tcp, [::]:80->8000/tcp   courses-api
    courses-api
    courses-api
    7c69c31d472f39b9b1f3c9ea869e3af8db6dd89d4e846f5ecf72823187ff3f5e
    CONTAINER ID   IMAGE                          COMMAND                  CREATED                  STATUS                  PORTS                                     NAMES
    7c69c31d472f   se93jmh0/week6-courses-api:1   "uvicorn main:app --…"   Less than a second ago   Up Less than a second   0.0.0.0:80->8000/tcp, [::]:80->8000/tcp   courses-api

    2026-Nov-18 19:31:40 UTC student@lab-5
    0 314 ~/week6-fastapi-docker
    $ 

### Re-Querying the Course

E.g.:

    2026-Nov-18 19:31:40 UTC student@lab-5
    0 314 ~/week6-fastapi-docker
    $ curl -i http://127.0.0.1/courses/4
    HTTP/1.1 200 OK
    date: Wed, 18 Nov 2026 19:33:32 GMT
    server: uvicorn
    content-length: 99
    content-type: application/json

    {"code":"ACC29","title":"Advanced Cloud Computing [2029 edition]","lecturer":"Donald Knuth","id":4}
    2026-Nov-18 19:33:32 UTC student@lab-5
    0 315 ~/week6-fastapi-docker
    $ 

### Cleaning up

    docker stop courses-api
    docker rm courses-api

...and if the volume (i.e., the Courses database) is no-longer required:

    docker volume rm week6-courses-data
