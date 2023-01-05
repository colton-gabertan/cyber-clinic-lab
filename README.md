# Requirements
- Debian-based VM, preferably Ubuntu Desktop 20.04 LTS
	- At least 4GB of RAM
	- 80GB of Storage
	- Internet connection
 - Docker
	 - docker-ce
	 - docker-ce-cli
	 - containerd.io
	 - Docker Compose (from the official repo)
- Cloned Githup Repo
	- https://github.com/colton-gabertan/cyber-clinic-lab
- Splunk Account
	- https://www.splunk.com/en_us/sign-up.html?redirecturl=https://www.splunk.com/

# Usage
To begin, please ensure that you already have a Splunk Account as it is required to install an application manually that we will be using for our labs.
> **note**: Be aware that this lab is volatile in its current state, so a lot of the data we generate will be lost. It is designed this way on purpose to not bloat your systems while engaging with the training program. 

### Cloning the Repo

The first thing you need to do is clone the github repo listed above onto your virtual machine.

The command to do so is:
```
git clone https://github.com/colton-gabertan/cyber-clinic-lab.git
```

After cloning, navigate into the repo. 
```
cd cyber-clinic-lab
```

Within this directory is where we may run our `Docker Compose` commands in order to spin up the containers for use and also to spin it back down to reclaim your system's resources when done. 

### Essential Docker Commands

Essentially, I will only require you to know three commands. The first command will go ahead and download the container images from the internet, build them to the specifications and run them locally. 
> **note**: Upon the very first time you run this command, expect it to take up to 10 minutes. It will also create output to your console to track the progress of the downloads, builds, and health of the cluster. Subsequent runs of this command will be able to spin up the entire lab in approximately one minute, thanks to Docker's caching feature.

```
docker compose up -d
```

The next command spins the containers down, and actually deletes them from memory for cleanup. If you wish to persist some of the data, run it without the flag.

```
docker compose down -v
```
> **note**: The `-v` removes a volume that is created for the lab. This volume will contain log data we will be generating; however, this data will usually not be needed outside of the exercises. So, it is best to delete the data by default as to not run out of storage.

This last command is important as it allows us to do administrative work *within* the containers and access them for the labs.

```
docker attach <container_name>
```

or alternatively:

```
docker exec -it <container_name> /bin/bash
```

Default container names:
- kali
- snort
- meta

This command grants us a shell into each container where we may use it as if we have an ssh connection. It is also very important to exit the shells gracefully via the exit signal:

```
Ctrl+p+q
```
> **note:** The Splunk instance is accessed via your browser so avoid connecting to it via the docker attach method. Also, not exiting via the control signal may actually stop executing the container depending on how you exited. 
