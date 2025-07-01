# DB2 rootless Docker image
## Intro
This unofficial DB2 image is rootless, and does not require `--privileged`. Built on and working with Podman, and DB2 v11.5.9. Docker I haven't actually tested but there's nothing that should cause issues. I also have not tested other versions of DB2

**Do NOT use this in production.** I created this for testing purposes only. It doesn't even have persistence, and I have no plans to implement it.

Hope someone finds this useful

## Credits
I did not create the entrypoint script, I took the one from [this project](https://github.com/Taskana/taskana/blob/master/docker-databases/db2_11-5/entrypoint.sh) - I modified it a little to work with my image. See the entrypoint.sh file itself for more information.

I am not affiliated with them, or IBM.

## Disclaimer
This project is provided "as is" and without any warranty or guarantee of any kind. Use it at your own risk.

The author is not responsible for any damage, data loss, system failure, or other issues that may result from using this software.

See the Apache License 2.0 for more information.

## How to build
### Step 1
Download a tar archived installer for DB2 from IBM. I will not be providing this, you need to bring your own, and then edit the Dockerfile accordingly. I recommend getting version v11.5.9. Verify the name matches the one in the Dockerfile

### Step 2
Pick the base you want (currently only Rocky Linux 9), change to the directory accordingly. Move the DB2 installer herre. 

Run:
```
docker build --tag localhost/db2-rootless:latest .
```

(Optional) After build is finished, clear the dangling images left over from the build. I recommend doing this, as the images are in the gigabytes
```
docker image prune 
``` 

### Step 3
```
docker run -d --name mydb2 -p 50000:50000 -e LICENSE=accept localhost/db2-rootless:latest 
```

This creates a fully barebones install, you're gonna have to make a db and anything else by yourself.
You can test if it's working by running this in the host system:
```
docker exec -it mydb2 /bin/bash
```
And in the container, run:
```
db2 create database
db2 list db directory
```

### Step 4
Enjoy (not in production please)