# Docker Case Study(DCS)

## Problem Statement
- Multiple Allocation of Linux systems for users
- Each user should have independent Linux System
- Specific training environment should be created in Container
- Monitor participants containers
- Automate container creation and deletion.

## Solution

We will be using the `ubuntu` image for multiple containers.

### Allocate containers to Users
- Now we can allocate our containers to Users

- To create a container for a single user with user id `user` the command is: `docker create -it --name user training:v1 /bin/bash` . This creates a container with name of `user` from the image `training:v1`, but doesn't start the container.

- If we are given a file with the list of user ids then we can automate the
allocation of containers. Let the file containing user ids be `userfiles`:

```
user1
user2
user3
user4
user5
```

- The script that automates container allocation is `allocate`:

```bash
#!/bin/sh

file=$1

while read user || [[ -n "$user" ]]
do
	container_id="$(docker create -it --name $user training:v1 /bin/bash)"
	echo "$user: $container_id"
done < "$file"

```

We can run the command `allocate userfiles` to allocate a container for every
user id in `userfiles`.

### Using an container
- Now that the container has been allocated for each user we need to start it
and attach the container (attach local standard input, output, and error streams
to the running container).

- The container for user with id `user` can be used by running the following
commands:

```bash
docker start user # Starts container abc
docker attach user # Attaches local stdin, stdout and stderr to abc
```

Running the above commands leads to the user getting access to the shell of his
"system".

The user can use the `exit` command to exit from the container. This also stops
the container but the state is preserved. Next time the container is started
it reloads the previous state.

### Monitoring containers
-	We can view the resource usage of container `user` by running the command
	`docker stats user`. This gives an output of the following format:

	```
	CONTAINER ID        NAME                CPU %               MEM USAGE / LIMIT   MEM %               NET I/O             BLOCK I/O           PIDS
	aa4fb37ade87        traincont           0.00%               716KiB / 1.952GiB   0.03%               898B / 0B           0B / 0B             1
	```

-	We can also view the logs of container `user` by running `docker logs user`. We
	can get the real time logs by running `docker logs -f user`. This follows the log
	output and prints every update.

### Deleting containers
Exiting from an attached container only stops the container. But the container
state is still preserved and it can be reloaded easily.

To completely remove the stopped container `user` we can use the
`docker rm user` command.

If we have a file containing the list of containers to remove we can automate it
with the script `deallocate`:

```bash
#!/bin/sh

file=$1

if [ -z "${file}" ]
then
	echo "Usage: delete <file>"
	exit
fi

while read user || [[ -n "$user" ]]
do
	docker rm $user
done < "$file"

```

We can use the above script by running `./deallocate userfiles` (`userfiles` has
been described earlier).

### Conclusion

Thus we see that containers provide an efficient way to manage isolated
systems and make it easy to monitor the systems. Isolation provides safety to the
host system and other containers.
