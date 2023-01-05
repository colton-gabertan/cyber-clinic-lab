# Cyber Clinic Docker Lab 0
Although a ton of the heavy lifting in spinning up infrastructure has been automated, we must run through each container to understand what exactly is going on. My goal for you is to familiarize you with this system to the point where you can customize it and make it your own.

This will also be beneficial should we roll out different infrastructures to accompany the future labs.

### Spinning up the lab
---
- [ ] **Read the usage guide and use the specified Docker command in order to spin up our infrastructure.**

Upon the first time you run this command, expect it to take approximately 10 minutes. Any subsequent runs will be exponentially faster, thanks to not having to dowload the dependencies and caching. Docker will be downloading each of the container images, and running the configurations needed in order to network them together on your machine. 

What is happening is it is:
- Creating a private subnet within your network to host Kali Linux and Metasploitable2
	- Downloading a custom suite of the Kali Linux tools to reduce bloat
- Creating an interface that is monitored by Snort
	- Configuring Snort properly and automatically
- Creating shared folders to forward logs to Splunk
- Spinning up Splunk's web interface which you will be accessing

- [ ] **Perform a Sanity Check**
	- [ ] Inspect the network
	- [ ] Inspect the container statuses

As you can see, there is output data to the console indicating that everything should be up and running; however, let's run a sanity check to ensure that everything is in place and functioning properly.

```
[+] Running 6/6
 ⠿ Network ids-net                       Created                           0.1s
 ⠿ Volume "cyber-clinic-lab_log_volume"  Created                           0.0s
 ⠿ Container kali                        Started                           5.6s
 ⠿ Container splunk                      Healthy                          65.4s
 ⠿ Container meta                        Started                           5.4s
 ⠿ Container snort                       Started                          65.4s
```

The first thing to check is if the network is allocated properly.

```
docker network inspect ids-net
``` 

should allow us to peek at the current configuration of the network. Your output should look like:

```json
[
    {
        "Name": "ids-net",
        "Id": "f2142922022e8742bd90f78d5e9f50d5c65b6dc22529b7cd87ee48702c1f47a1",
        "Created": "2023-01-03T11:18:12.078798749-08:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.18.0.0/16",
                    "Gateway": "172.18.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "56c43cb5d414a87fb8593a339723e1d686edc073ba33c6c36d5e4b5f5ad41017": {
                "Name": "meta",
                "EndpointID": "32b8101452b13c4f61d33d7c78a7754163727e9e48d1f480d8b7438f609a3460",
                "MacAddress": "02:42:ac:12:00:03",
                "IPv4Address": "172.18.0.3/16",
                "IPv6Address": ""
            },
            "933c826b485a0bb4dc00e0f9149c687d0f2d8035361cac55fe3d60cb5eb069cb": {
                "Name": "kali",
                "EndpointID": "bd46edef62907e59db8224434ab2c7bc2b20cb0887edc06d881dee7548dd97f5",
                "MacAddress": "02:42:ac:12:00:02",
                "IPv4Address": "172.18.0.2/16",
                "IPv6Address": ""
            }
        },
        "Options": {
            "com.docker.network.bridge.name": "ids_bridge"
        },
        "Labels": {
            "com.docker.compose.network": "ids-net",
            "com.docker.compose.project": "cyber-clinic-lab",
            "com.docker.compose.version": "2.13.0"
        }
    }
]
```

The main things to look out for are that your subnet corresponds to the one listed above, and that the `kali` and `meta` containers are on this subnet with the same IP addresses listed above.

You also want to ensure that the `"com.docker.network.bridge.name": "ids_bridge"` line is present. This is the interface that our Snort container listens to in order to collect traffic between the Kali container and the Metasploitable one. 

To check the statuses of the containers, run:

```
docker container ls
```

Your output should be similar to:

```
CONTAINER ID   IMAGE                             COMMAND                  CREATED         STATUS                   PORTS     NAMES
2e78115ed329   cyber-clinic-lab-snort            "/bin/sh -c 'snort -…"   6 minutes ago   Up 5 minutes                       snort
56c43cb5d414   tleemcjr/metasploitable2:latest   "sh -c '/bin/service…"   6 minutes ago   Up 6 minutes                       meta
85a7db0000ad   cyber-clinic-lab-splunk           "/sbin/entrypoint.sh…"   6 minutes ago   Up 6 minutes (healthy)             splunk
933c826b485a   cyber-clinic-lab-kali             "/bin/bash"              6 minutes ago   Up 6 minutes                       kali
```

Please ensure that everything is Up and Healthy before proceeding.

### Intro to Snort
---
This lab is mainly focused on exploring the environment, while the subsequent ones will be designed for more specific tasks and learning.

For now, we will learn how to actually use Snort with the configuration I have set. Essentially, Snort lives on the same network as the VM you're using to host the cluster, while it has full visibility into the private network that Kali and Meta live on. 

- [ ] **Open a shell into the Snort container**

Via the usage guide, *attach* to the `snort` container:

```
docker exec -it snort /bin/bash
```

From the CLI you now have a shell inside of the snort container.

- [ ] **Explore the config file**

Navigate to `/etc/snort/snort.conf`. To view the file, I've installed either vim or nano as the default text editors within this container. 

Lines 44-48 of the config file should read:
```
# Setup the network addresses you are protecting
ipvar HOME_NET 172.18.0.0/16

# Set up the external network addresses. Leave as "any" in most situations
ipvar EXTERNAL_NET any
```

These snort variables tell us that it is monitoring HOME_NET and will consider anything else an EXTERNAL_NET for alerts. Snort alerts will also be set up for within the HOME_NET, but they will have another label if an IP address from outside the subnet is able to access the containers within our HOME_NET.

Line 182 should show us the path where our logs will be written to:
```
config logdir: /var/log/snort
```

We may collect the binary packets themselves (PCAPs/ TCPDUMPs) as well as view the raw alert logs that are forwarded to Splunk.

The last thing to note is that we may also specify which rules are applied to the network monitoring. By default I've turned off the community ones and am only allowing local rules to be ran. This is done on purpose for a signature-writing lab later on.

Starting at line 535, you may view the extensive list of signatures that you may enable if you wish to tinker with the community rules. However, please ensure that the line containing `local.rules` is not commented. This file is where our custom signatures live.

To view the few rules already set as an example, view the file `/etc/snort/local.rules`

- [ ] **Start the Snort service**

By default, the Snort service is not running. We must run it from the CLI in order for it to begin collecting and monitoring network traffic.

To begin and troubleshoot initially, run:
```
snort -q -A console -i ids_bridge -c /etc/snort/snort.conf
```

For now, we are going to output our alerts to the console so we can check if it's running properly.

- [ ] **Check and troubleshoot Snort**

Spawn a new shell on the docker host so that we may connect to the Kali machine and generate a bit of network traffic. If working properly, you should see the alerts be output to the snort container in real time.
> To spawn a new shell, click out of the terminal and run `CTRL+ALT+T` . We will use the second terminal window for the Kali access.

Connect to the kali container via:

```
docker attach kali
```

And it should drop you in as a root user. We will generate some simple traffic by pinging the Metasploitable container.

From the kali container run:
```
ping -c 2 meta
```

This should generate an alert on the snort console in real time. If you are running the windows side-by-side you should see it create the alert after running the ping.

<img src="/screenshots/Pasted image 20230103120853.png">

- [ ] **Restart the Snort Service**

Now that the sanity check has been performed, we should alter the way that Snort is running so that it generates the logs to be forwarded to Splunk. This is as simple as running the snort command with slightly different flags:

```
snort -q -A full -i ids_bridge -l /var/log/snort -c /etc/snort/snort.conf
```

This is important as it will format log data in a better way for Splunk to digest, and we may leave the container running like this in the background and trust that it is working correctly.

Exit the snort container via:
```
CTRL+p+q
```
> You may also exit the kali container in the same manner.

### Intro to Splunk
---

The second container I suspect most students may not be familiar with is the Splunk instance.  Both within our lab and in environments where you'd typically see Splunk, we access this service via our web browser.

- [ ] **Access the Splunk Service**

This Splunk container lives on the same network as the host, just like the Snort service. This is supposed to emulate a bit of network segmentation as our management containers, Snort and Splunk, should be separate from the test environment (Kali & Metasploitable). 

Since it is on the host's network, the web interface may be accessed from the browser. It serves it over port 8000. Visit it via the URL:

```
localhost:8000 
```

You will be greeted by an administrative login page. By default the username is `admin` and you may find and even set your own password in the docker-compose.yml file. I've defaulted it to `password`.

![[Pasted image 20230105094558.png]]

- [ ] **Install the Snort for Splunk App**

One of the neat things about Splunk is that people may install and even develop applications that allow us to visualize and process all of the aggregated log data in order to gain the most effective insights about our environment.

Luckily, Splunk plays very nicely with Snort and we can download a dashboard that we can use to see and process our alerts.

From the homepage, click on `+ Find More Apps` and search for `Snort Alert for Splunk`

<img src="/screenshots/Pasted image 20230105094922.png">

Go ahead and install the application. You will be prompted to enter the credentials for the Splunk account you should have created previously as per the Usage Guide. After installation is complete, the app may be accessed anytime from the left navigation bar on the homepage.

- [ ] **Generate some alerts to visualize**

Upon a clean install, there will be no data to visualize, so let's generate a bit of data that may reflect an attacker scanning your environment. Then, we will review how that looks in Splunk.

Open a terminal and attach to the kali container via the command we previously used. We are going to then perform a standard `nmap` scan of the vulnerable container.

```
nmap meta
```

This will set off a few of the default alerts I have left configured for Snort, and the output from the kali container should look something like:

```bash
└─# nmap meta
Starting Nmap 7.93 ( https://nmap.org ) at 2023-01-05 17:54 UTC
Nmap scan report for meta (172.18.0.3)
Host is up (0.0000070s latency).
rDNS record for 172.18.0.3: meta.ids-net
Not shown: 979 closed tcp ports (reset)
PORT     STATE SERVICE
21/tcp   open  ftp
22/tcp   open  ssh
23/tcp   open  telnet
25/tcp   open  smtp
80/tcp   open  http
111/tcp  open  rpcbind
139/tcp  open  netbios-ssn
445/tcp  open  microsoft-ds
512/tcp  open  exec
513/tcp  open  login
514/tcp  open  shell
1099/tcp open  rmiregistry
1524/tcp open  ingreslock
2121/tcp open  ccproxy-ftp
3306/tcp open  mysql
5432/tcp open  postgresql
5900/tcp open  vnc
6000/tcp open  X11
6667/tcp open  irc
8009/tcp open  ajp13
8180/tcp open  unknown
MAC Address: 02:42:AC:12:00:03 (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 0.27 seconds
```
> **note:** This process is called enumeration. It is the reconaissance phase of an attack where one may attempt to gather information as to the possible vectors that may be exploited. Metasploitable2 is a service that is purposely very vulnerable and is used for learning exercises such as this one.

- [ ] **Review the alerts in Splunk**

After running the scan, head back to the Splunk instance and review how the dashboard not only visualizes the data, but also grants some insights such as which signature was hit, the frequency, the event sources, etc. 

<img src="/screenshots/Pasted image 20230105121532.png">

### Conclusion
---

With this lab you should now be familiar with each default container and how to access them. The following labs will build upon this material as we begin to explore more granular topics pertaining to enterprise security. 

- [ ] As per the usage guide, spin down your lab

```
docker compose down -v
```


