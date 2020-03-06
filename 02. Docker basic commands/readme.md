# Hands-on labs Introduction to Docker

## Exercise 02: Docker basic commands
In this exercise we are going to navigate through a number of basic Commands

- Get all running containers on you system
```
docker ps
```

- Get all running and stopped containers on your system
```
docker ps -a 
```

> Can you see two images that have exited?

- Finding images on your local machine
```
docker images
```

- Finding images on docker hub
```
docker search microsoft
```
> There are a lot more Linux containers available. Checkout https://hub.docker.com to find base images. Right click the whale icon in your tray to switch to Linux containers and try docker search <product> again. E.g. docker search   

- Running a container in the background (-d)
```
docker run -d mcr.microsoft.com/windows/servercore:1809
```

- Running a container interactively (-it)
```
docker run -it mcr.microsoft.com/windows/servercore:1809
CTRL-P CTRL-Q
```

- Attach to a running container
```
docker attach <name of container>
```

- Start a new process in an existing container
```
docker exec <name of container> cmd
```

- Inspect properties of container
```
docker inspect <name of container>
```

- Rename a container images
```
docker tag <name of container image> <tagname>
```
- Pull (download) a container images
```
docker pull microsoft/aspnet
```
 