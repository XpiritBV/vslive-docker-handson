# Hands-on labs Introduction to Docker

## Exercise 03: Build a new Container Image
In this exercise, you will go through the steps of creating a new Container image. First from the command line using Docker commit and in a second step by using a docker file and the docker build command. Accompanying files can be found I this folder. 

### Using steps on the command-line

- Run a new container, add some files and exit the modified container
```
docker run -it --name mycontainer mcr.microsoft.com/windows/servercore:1809 cmd
md builddemo
cd buidldemo
copy con myfile.txt
this is a text file
ctrl+c
exit
```

- Get all running and stopped containers
- Create a new images by committing a new layer 
- Run the new container

```
docker ps -a
docker commit mycontainerimage mycontainer 
docker images 
docker run -it --name myowncontainer mycontainer cmd
dir
exit
```

- If everythin went well, the file you created earlier is now part if the new container

### Using the docker build file
- On your command line ensure you are in the folder 02. Build a new Container Image,  that contains docker file (Dockerfile)
- Build the docker image from the Dockerfile
- Find all images on your machines
- Find the running container ID
- Find the IP address of the running container

```
Docker build -t mynewiiscontainer .
Docker images
Docker run -d -p 80:80 mynewiiscontainer 
Docker ps 
Docker inspect containerID
```

- Start a browser and navigate to the IP address and see the new website running on port 80 serving the index.htm file that is in the current folder
