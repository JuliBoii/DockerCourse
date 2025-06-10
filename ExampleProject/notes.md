## Adding Files to Docker Container

```dockerfile
COPY ./ ./
```

- The first "./" path, is the path to the folder to copy from on *your machine*
    - the path is relative to current build context, i.e., the current directory we are working in
- The second "./" path, is the path to copy stuff to inside *the container*

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

