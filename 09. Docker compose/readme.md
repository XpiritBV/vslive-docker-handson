# Hands-on labs Introduction to Docker

## Exercise 09: Using Docker-compose to orchestrate multiple containers
In this exercise we want you to experiment with Docker-compose. It is great that you can run one container, but what if you need more

* First clone the github repository https://github.com/XpiritBV/vslive2020-cw in a directory on your local machine (e.g. c:\temp\)
* Open the solution in Visual Studio and build with Visual Studio 

You can start the solution, but it will not run. There is no database, and there are dependencies in the projects. Let's try to solve that with Docker
* Switch your Docker client to use Linux containers by using the icon in the tray. Right click the whale icon and select Switch to Linux containers) 

## Run a SQL Server container

Instead of running SQL on your local machine, a convenient and fast way to have new and fresh database is to run SQL in a container. There are standard images for that. Find the image on Docker hub and start the container

- Navigate to [Docker Hub](https://hub.docker.com) and search for SQL Server
- You will find this page [https://hub.docker.com/_/microsoft-mssql-server](https://hub.docker.com/_/microsoft-mssql-server)
- Start the SQL Server container as described on Docker hub
> pay attention that the environment variables need to be enclosed in double quotes " " on windows

```
docker run -e "ACCEPT_EULA=Y" -e "SA_PASSWORD=Pass@word" -p 5433:1433 --name sqldocker -ti mcr.microsoft.com/mssql/server:2017-latest
```
- (Optionally) Use Visual Studio Server Explorer to connect to the SQL Server using localhost, 5433
- Modify the appsettings.json of the Leaderboard.webapi project to point to localhost,5433 and run the project. Try to run the Get scores and test the results

## Running the application in containers
Now we are going to run the application in a container. Or actually, 2 containers. 1 for the API and 1 for the UI

- Open a command line and navigate to the **root* of your solution. (where the sln file resides)
- Build the Docker file from this location

```
docker build -t vslive/leaderboard.webapi -f src/Services/Leaderboard.WebAPI/Dockerfile .
```

- Run the container.
```
docker run -d -p 5000:80 vslive/leaderboard.webapi
```

- visit website on localhost:5000 and see that it fails. There is no SQL accessible
> What do you think is wrong?


- Change name of SQL server in connectionstring to sqldocker (or the generated name of the SQL container)
- Build and run the container again

## Create or use compose file
Now it is time to fit everything together. We want to start the UI, the API and the SQL Server together. Take a look at the compose file in the root directory.

- code docker-compose.yml
> Read through all the settings and see what it means 

- Start the composition
```
docker-compose up
```
>Does the SQL Server connection work?
>what do you need to change?
>You can access the containers from the same compose file by its name

-Bring down the app
```
docker-compose down
```
>or hit CTRL-C in the terminal

## Scaling up
Your website is running. Assume that you want to have more instances of the UI. You can scale up very easily with compose as well.

- in your root directory run the scale command
```
docker-compose up --scale gamingwebapp=10
```
>But how do you balance the load?

