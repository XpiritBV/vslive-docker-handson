# Hands-on labs Introduction to Docker

## Exercise 04: Windows and Hyper-V Containers

Windows can run containers in 2 ways. The normal "shared kernel" way, like on Linux, and the more isolated way, Hyper-V Containers. In below exercises you will see the differences

> Read more here https://docs.microsoft.com/en-us/virtualization/windowscontainers/manage-containers/hyperv-container

- Start a normal windows container
- Look at the running processes, take note of the PID
```
docker run -d --isolation process mcr.microsoft.com/windows/servercore:1809 ping localhost -t
docker top [containerid]
```

- Start a hyper-v container
```
docker run --isolation hyperv -d mcr.microsoft.com/windows/servercore:1809 ping localhost -t
docker top [containerid]
```

- start a powershell command promopt and get process on your local machines
```
get-process -Name ping
get-process -Name vmwp
```
- Take note that the PID of ping is actually the PID in the shared kernel container. The Hyper-V container is running isolated in the vmwp process
