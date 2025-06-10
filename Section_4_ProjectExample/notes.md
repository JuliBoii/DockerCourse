## Adding Files to Docker Container

```dockerfile
COPY ./ ./
```

- The first "./" path, is the path to the folder to copy from on *your machine*
    - the path is relative to current build context, i.e., the current directory we are working in
- The second "./" path, is the path to copy stuff to inside *the container*

---

## _Interacting with Node Server_

- Our container has its own set of ports that can recieve traffic
    - But by default, no incoming traffic to your computer is going to be directed into a container
- To ensure it can interact
    - We need to set up an explicit port mapping
- Port Mapping
    - Any time that someone makes a request to a given port to the local network
    - Take that request and automatically foward it to some port inside the container
- **_This is only talking about incoming request_**
    - By default, our docker container can make requests on its own behalf to the outside world
        - We see that anytime we are installing a dependency
        - It reaches out to the internet
        - There is no limitation to reach out
    - Strictly limits incoming traffic for the container
- To do this we need to add commands to our run command
    - Port fowarding is strictly a runtime constraint

```commandline
docker run -p <port 1>:<port 2> <image ID/Name>
```
- Port 1
    - Route incoming requests to this port on local host to...
- Port 2
    - this port inside the container
- **_They do not have to be the same port/identical_**
- We could also change the port traffic is being directed to, so long as our application is updated with the new port

---

## Running a container with a shell

- To run a container with an interactive shell we use the following command:

```commandline
docker run -it <image ID/Name> sh
```

- We are overriding the default startup command with `sh`
    - It will start up a shell (the shell program) that we can type commands into inside the container
- We did not specify the ports because we only want to interact with container

- When we interact with the container, we see that the files we copied into the container are in the root directory
    - **_This is not best practice_**
        - For the reason that,
            - if we happen to have any files or folders that conflict with the default folder system (which is likely)
            - we might accidentally overwrite some existing files or folders in the container
            - **Which is not ideal**
- Thus we need to make a change to our `Dockerfile`
- Rather than copying everything into the root project directory
    - We will copy everything into a seperate, nested directory

---

## Specifying a Working Directory

- We add the `WORKDIR` instruction to our Dockerfile
    - Pass a reference to a folder to it
    - Any following command/instruction in our Dockerfile
        - will be executed, relative to this folder

```dockerfile
WORKDIR /path/to/directory
```

- We should then rebuild the file after updating the `Dockerfile`
- Using the `docker build` command in the terminal

---

## Interacting with running container

- While we have our server running
- In a seperate terminal tab
    - We obtain the container ID using the following command

```commandline
docker ps
```

- We copy the container ID
- then we can attach to that container and start up a shell by using:

```commandline
docker exec -it <Container ID> <program>
```

- So we need to specify the container
- And the program we want to run
    - For this instance we want to run a shell so we add `sh` for `<program>`

- We notice, for this case, that we are in the directory we specified in our `Dockerfile`
- The `WORKDIR` command not only affects the instructions following its instance
- It also affects commands that are executed inside the container later on
    - _Through the `docker exec` command_

---

## Updating our files, With running container

- We notice that if we change anything in the files for our project
    - while the container is running
    - the container does not update
- Anytime we create the image or container
    - we are taking a snapshot of the file system
    - So we are taking a snapshot of the following files for this case
 ```commandline
./index.js
./package.json
```

- So our application is running based of the old version of index.js
    - not our updated file
- If we want to update the file in the container
    - We need some configurations, **_that will be covered later_**
- For this example
    - We will have to completely rebuild the container
    - We notice, when we rebuild
        - That every instruction after the `COPY` command will be re-executed
        - Which is not ideal

---

## Minimizing Cache Busting and Rebuilds

- For this example
    - We will split the `COPY` instructions
    - Since we know that the `index.js` file will be modified more often than our `package.json` file
    - We move the `COPY ./index.js` instruction to after the installation of our dependencies
    - Thus, we will not have to reinstall our dependencies each time we modify `index.js`
- In conclusion
    - The order of instructions **does matter**
    - So it is nice to segment out the copy operations to make sure we are only copying the bare minimum for each successive step