# Hands-on labs Introduction to Docker

## Exercise 01: Run a docker container on Windows
In this exercise you will run a container to inspect the concepts of immutability. For that we are going to use Docker.

- Open a commandline 
- Verify that docker is correctly installed by running `docker version`. 
- Check that the docker **server** OS is Windows

```
Server: Docker Engine - Community
 Engine:
  Version:          19.03.5
  API version:      1.40 (minimum version 1.24)
  Go version:       go1.12.12
  Git commit:       633a0ea
  Built:            Wed Nov 13 07:36:50 2019
  **OS/Arch:          windows/amd64**
  Experimental:     false
```
- If it is not windows, but Linux, right click the Docker whale in the tray, and select [Switch to Windows Containers] 

Start by running a simple windows container with the command prompt in it.

```
docker run -it mcr.microsoft.com/windows/servercore:1809 cmd
```

- Run the following commands inside the newly started container 

```
dir
md demo
cd demo
copy con myfile.txt
Hello this is in my docker container
ctrl+c
dir
```

- Now you should see the text file you created in the steps above
- Exit the container 
```
ctrl+p ctrl+q
```

- Check all running containers

```
docker ps
```

- Now you should see a container running with a generated name
- Verify that you don’t see the file on your host by going to c: and running dir. 
- Stop the running container

```
docker stop name-of-the-container
```

- Check all running containers

```
docker ps
```

- View all running and stopped containers 

```
docker ps -a
```

- Start a new windows container and validate that the file you created earlier is not in there

```
docker run -it mcr.microsoft.com/windows/servercore:1809 cmd
dir
```