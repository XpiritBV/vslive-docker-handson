# Hands-on labs Introduction to Docker

## Exercise 06: Volume mappings to modify files in the container
Files reside in the container. You cannot open them on your local machine. But how to deal with data that you want to keep outside of the container, or if you want to use an IDE to change files. You can use volume mappings for that

- Run a new container with the current directory as volume mapping (%CD% works only in the CMD and not in powershell!)
```
docker run -ti -v %cd%:c:\data mcr.microsoft.com/windows/servercore:1809
```

- Modify the index.html file outside the container
- Check contents of file in container
```
type index.html
```
