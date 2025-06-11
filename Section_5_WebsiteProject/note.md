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
/Section_5_WebsiteProject/docker-compose.yml
```

- **_Upon further reading, I have learned that Docker Compose V2 changed the perfered naming convention for
  docker-compose files_**
- docker-compose files should now follow the following convention:

```commandline
/Section_5_WebsiteProject/compose.yml
```

- Both naming filenames are accepted
    - But `compose.yml` is recommended


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
    - **Upon further use, I have come to find out that Docker Compose V2, made this field redundant**
    - Specifies the version of docker-compose to use
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
docker stop <container ID>
```

- Now with `docker-compose` we have multiple containers running, but what do we do if we want to stop them
    - We do not want to constantly input `docker stop` with each container id


- To launch a `docker-compose` in the background we add:

```commandline
docker-compose up -d
```

- To stop the running containers using the following command

```commandline
docker-compose down
```

- Testing the commands results in the following:

```commandline
Section_5_WebsiteProject % docker-compose up -d
[+] Running 3/3
 ✔ Network section_5_websiteproject_default           Created                          0.1s
 ✔ Container section_5_websiteproject-redis-server-1  Started                          0.4s
 ✔ Container section_5_websiteproject-node-app-1      Started                          0.5s
Section_5_WebsiteProject % docker-compose down
[+] Running 3/3
 ✔ Container section_5_websiteproject-redis-server-1  Removed                          0.2s
 ✔ Container section_5_websiteproject-node-app-1      Removed                          0.7s
 ✔ Network section_5_websiteproject_default           Removed                          0.2s
```

---

## Container Maintenance with Compose

- What do we do in the event that our containers crash or have an error
- There are multiple ways to mitigate and essentially restart a container when this occurs

### First Off

- Let us add some code into `index.js` so that
    - Our server crashes entirely everytime
        - Someone visits our root route

We will add the following to `index.js` after line 2:

```javascript
const process = require('process');
```

Then add the following after line 12:

```javascript
process.exit(0);
```

- Looking at the previous line, the `0` we pass to `process.exit` is a status code
    - By adding a zero, we are indicating that we just exited our node server, because we wanted to.
    - Meaning that everything is okay, we simply stopped the process because we wanted to
    - If we added in a status code other than `0`
        - That means our process exited, because an error occurred or something went wrong
        - **This is importanat to note, becuase that affects how Docker decides whether or not to restart our containers
          **

Then we test it the terminal to see the behavior, since we updated a file we will need to rebuild the compose file

We type the following:

```commandline
docker-compose up --build
```

Then navigate to our browser and open `localhost:4001` and notice that our website failed

Looking at the terminal, we see the following new line:

```commandline
...
node-app-1 exited with code 0
```

At this point, our running container and the software inside of it, has crashed

Opening a second tab in our terminal and entering `docker ps` we see only one container running:

```commandline
Section_5_WebsiteProject % docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS      NAMES
e02f8227b6a1   redis     "docker-entrypoint.s…"   6 minutes ago   Up 6 minutes   6379/tcp   section_5_websiteproject-redis-server-1
Section_5_WebsiteProject %
```

Now we will see how we can get docker-compose to automatically restart the crashed/stopped container

- We are going to specify something called a restart policy
    - inside of our docker-compose file
    - There are 4 different restart policies we have access to.

### Restart Policies

|      Code      |                                Description                                |
|:--------------:|:-------------------------------------------------------------------------:|
|      "no"      |     Never attempt to restart this . container if it stops or crashes      |
|     always     | If this container stops for *for any reason* always attempt to restart it |
|   on-failure   |          Only restart if the container stops with an error code           |
| unless-stopped |        Always restart unless we (the developers) forcibly stop it         |

### Always

- We update our `compose.yml` file to appear as such:

```yaml
services:
    redis-server:
        image: 'redis'
    node-app:
        restart: always
        build: .
        ports:
            - "4001:8081"
```

- We the re-run our compose file
- Going to `localhost:4001` again
    - We are met with the same crashed site
- Looking at the terminal
    - We see that the same output occurred, but this additional lines
- The output looking like the following:

```commandline
...
node-app-1 exited with code 0
node-app-1      |
node-app-1      | > start
node-app-1      | > node index.js
node-app-1      |
node-app-1      | Listening on port 8081
```

- This shows that the node container exited but restarted up again

### On-Failure

- We update our `compose.yml` file to appear as such:

```yaml
services:
    redis-server:
        image: 'redis'
    node-app:
        restart: on-failure
        build: .
        ports:
            - "4001:8081"
```

- We the re-run our compose file, **_taking note that we have left `index.js` as is_**
- Going to `localhost:4001` again
    - We are met with the same crashed site
- Looking at the terminal
    - We see that the same output occurred, when we did not have the `restart` option in `compose.yml`
- The output looking like the following:

```commandline
...
node-app-1 exited with code 0
```

- This is due to the nature of this restart policy
    - Since the error code is `0`
        - Meaning we exited the application because we wanted to
        - Not because an error occurred
    - Thus, docker-compose will not attempt to restart the container
        - Because the container did not fail
- In this case, the container will never try to restart
- So we have to change our exit code

- Let us update code in `index.js` so that
    - Our server crashes due to a failure

We will update the following to `index.js` on line 13:

```javascript
process.exit(<any Non-Zero Number>);
```

- Adding in a status code other than `0`
    - Means our process exited, because an error occurred or something went wrong
    - **So, docker compose should attempt to restart our container**

Then we test it the terminal to see the behavior

Since we updated a file we will need to rebuild the compose file

We type the following:

```commandline
docker-compose up --build
```

Then navigate to our browser and open `localhost:4001`

Looking at the terminal, we see the following new lines, after building:

```commandline
...
node-app-1 exited with code 0
node-app-1      |
node-app-1      | > start
node-app-1      | > node index.js
node-app-1      |
node-app-1      | Listening on port 8081
```

This shows that the node container exited but restarted up again, due to the error being some reason other than `0`

### Unless-Stopped

- We update our `compose.yml` file to appear as such:

```yaml
services:
    redis-server:
        image: 'redis'
    node-app:
        restart: unless-stopped
        build: .
        ports:
            - "4001:8081"
```

Then we test it the terminal to see the behavior

Then navigate to our browser and open `localhost:4001`

Looking at the terminal, we see the following new lines, after building:

```commandline
...
node-app-1 exited with code 0
node-app-1      |
node-app-1      | > start
node-app-1      | > node index.js
node-app-1      |
node-app-1      | Listening on port 8081
node-app-1 exited with code 100
node-app-1      |
node-app-1      | > start
node-app-1      | > node index.js
node-app-1      |
node-app-1      | Listening on port 8081
node-app-1 exited with code 100
node-app-1      |
node-app-1      | > start
node-app-1      | > node index.js
node-app-1      |
node-app-1      | Listening on port 8081
```

---

## Why would we ever want to use always vs on-failure

- It really comes down to the purpose one's Docker container
- In some cases
    - You might have a container that you _always, always, always_ want to make sure is running
    - A good example of this would be
        - A web server
            - If you are running some public web application
            - Chances are a hundred percent of the time you want that server to be available
        - So it would be expected to be utilizing the `always` restart policy
            - That way, you are always attempting to get your server up and running
- If you are running some type of worker process
    - Like some container that is meant to do some amount of processing on some file and then naturally exit
    - That would probably be a good use-case for the `on-failure` restart policy
        - Because that worker container might eventually finish its job
        - And when it completes that process, you probabaly do not want it starting back up
            - Since it finished its job

---

## Docker-Compose PS

- In previous instances
    - When we are running stand-alone containers
- We can see the running containers by inputting the following in a terminal

```commandline
docker ps
```

- We will be shown the running containers
- Similarly, we can do this for docker-compose as well
- Starting up our docker-compose file, we then run the following command, in a seperate terminal tab

```commandline
docker-compose ps
```

- Which shows the running containers inside of the docker compose file
- But, there is something we need to clarify
    - When we ran the command, we ran it from the same directory as the docker-compose file
    - That is because, the command makes docker-compose specifically look for a docker-compose file
        - Inside the current directory
            - If it finds one, it will read it
            - Then it will try to find all of the running containers on your local machine
                - That belongs, essentially, to this docker-compose file
- If you try to run this command, from any other directory
    - You are going to get an error message
    - Essentially, telling you it cannot find any containers
- Outputting the following:

```commandline
% docker-compose ps
no configuration file provided: not found
```

- Docker-compose will not magically figure out which containers you are talking about
    - It needs the `docker-compose.yml` file as _kind of_ a reference to understand which specific containers its
      suppose to get the status of.