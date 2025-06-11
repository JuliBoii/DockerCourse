## Running a Container with a seperate Container running Redis

- When be initially build the docker file
    - Our working directory will look like:

```commandline
/app/index.js
/app/package.json
```

- When we run a container we will get the following output

```commandline
docker run <Docker Username>/<image tag/ID>
```

```commandline
> start
> node index.js

Listening on port 8081
node:events:485
      throw er; // Unhandled 'error' event
      ^

Error: connect ECONNREFUSED 127.0.0.1:6379
    at TCPConnectWrap.afterConnect [as oncomplete] (node:net:1637:16)
Emitted 'error' event on RedisClient instance at:
    at RedisClient.on_error (/app/node_modules/redis/index.js:406:14)
    at Socket.<anonymous> (/app/node_modules/redis/index.js:279:14)
    at Socket.emit (node:events:507:28)
    at emitErrorNT (node:internal/streams/destroy:170:8)
    at emitErrorCloseNT (node:internal/streams/destroy:129:3)
    at process.processTicksAndRejections (node:internal/process/task_queues:90:21) {
  errno: -111,
  code: 'ECONNREFUSED',
  syscall: 'connect',
  address: '127.0.0.1',
  port: 6379
}

Node.js v24.2.0
```

- Our application is trying to start up
    - But there is no Redis server running for it to connect to
- So we will be setting up a seperate container running Redis
- Which is straightfoward using the following command:

```commandline
docker run redis
```

- This starts up our Redis server
- We will now open a second terminal window
    - Inside here we will try running the container
    - Run the previous run command
    - Which results in the same error as previously
        - Despite the Redis container running
- The problem?
    - Even though both containers are running
    - There is no communication between the two containers
        - They are two absolutely isolated processes
- We need to set up some networking infrastructure between the two
- To do this we have Two options
    - Make use of the Docker CLI `docker`
        - There is an issue though
            - It is a real pain in the neck to do
            - It involves a handful of different commands that have to be re-ran every single time
            - In general, the CLI is not mainly used in industry
    - The more common approach
        - making use of a seperate CLI tool called
            - Docker Compose `docker-compose`
        - This is a seperate tool that gets installed along with Docker

```commandline
docker-compose
```

- When we run it, we will see the following:

```commandline
Usage:  docker compose [OPTIONS] COMMAND

Define and run multi-container applications with Docker

Options:

[...other text...]

Run 'docker compose COMMAND --help' for more information on a command.
```

- This is a seperate CLI from `docker`
- The relationship between the two is confusing, but is best generalized as:
    - Docker-Compose exist to keep from having to write out a ton of different repetitive commands with the Docker CLI
        - avoiding annoying tiny little options every time you want to start up a container
    - The Other thing Docker-Compose will do is
        - make it easy & very straightfoward to start up multiple docker containers at the same time.
        - Plus, automatically connect them together with some form of networking
            - Happening all behind the scenes from us, automatically
    - **_In general, docker-compose functions the same as the docker CLI, but allow you to issue multiple commands much
      more quickly_**

---

## Docker Compose Files

- To use `docker-compose` we are essentially gonna take the same commands that we were running before
    - But we are going to _kind of_ encode these commands into a very special file in our directory called:

```commandline
./Section_5_WebsiteProject/docker-compose.yml
```

- _To be clear_
    - We are not just copying & pasting our previous commands into the file
    - They will be written using a special syntax inside the YAML file

### Quick Preview of docker-compose.yml

- The containers I want created:
    - Container built tagged `redis-server`
        - Made using the `redis` image
    - Container built tagged `node-app`
        - Made using the Dockerfile in the current directory
        - Add some ports that we want designated

```yaml
version: '3'
services:
    redis-server:
        image: 'redis'
    node-app:
        build: .
        ports:
            - "4001:8081"
```

- `version` is needed for this file
    - it specifies the version of docker-compose to use
- `services` defines the 'services' we want docker-compose to make
    - i.e., creates the containers we want
    - `redis-server` & `node-app` is the name we want to tag the containers as
        - `image` specifies the image we want a container to be built on
        - `build` looks for a dockerfile, `.` tells the instruction to search the current directory
        - `ports` will designate the ports we want to connect
- **_We may notice that we are not specifying that we want our services to interact with each other_**
    - Just by defining these two services, inside the file
    - Docker-compose is going to automatically create both these containers
        - On essentially the same network
            - Having free acces to communicate to each other
            - In any way they please

---

## Starting up Docker-Compose

- Previously, when starting up a container we would write:

```commandline
docker run <image tag/ID>
```

- Instead, we will run the following for `docker-compose`

```commandline
docker-compose up
```

- Another thing to take note of is that the previous work flow involved
    - Building the docker image
    - Then running a container of the image
    - As such:

```commandline
docker build .
docker run <image tag/ID>
```

- But with docker-compose, this can be combined into one command

```commandline
docker-compose up --build
```

- Now when we run, we can see both containers running

---

## Stopping Docker-Compose Containers

- Previously, when we wanted to stop a container that was running in the background
- We would use the following command:

```commandline
docker stop <image ID>
```

- Now with `docker-compose` we have multiple containers running, but what do we do if we want to stop them