# 2022 Cyber Clinic Training Lab

This lab is a containerized private network that contains tooling commonly found in enterprise cybersecurity. It is used as a training resource for members of the UNLV Free Cyber Clinic. At its base, this lab contains:

* Splunk Enterprise
* Snort
* Kali Linux
* Metasploitable2

Scaling up and down is as simple as editing the `docker-compose.yml` file. Iterations of this lab will be created with variations of containers to emulate different scenarios.

## Dependencies:

> * A Debian-based virtual machine
> * Docker and Docker Compose


## Use Cases

### Case 1: Learning Enterprise Tooling

My goal is to enable students to explore a bit of both sides of security. Starting with Snort, students or home-labbers are set up to monitor the network promiscuously. This opens up possibilites such as signature-writing for IDS/IPS/Firewall rules, observing specific exploits, and learning how real indicators of compromise may look. 

With Splunk, you are able to analyze logs produced by attacks or suspicious events. You may develop dashboards and applications that help visualize large amounts of data and really practice the more analytical side of blue teaming. 

Integrating the Kali Linux container is what offers a good intro to red teaming or offensive security. The vulnerable Metasploitable machine plays hand-in-hand with this, allowing people to focus on learning their tools and being able to understand what exactly is going on when they run exploits from frameworks. 

This environment is supposed to provide abstraction in the setup; while offering a peak beyond the abstraction when it comes to the technicals of infosec. In essence, all of the heavy lifting of spinning up infrastructure is automated, while allowing creative freedom to explore infosec topics and collect data to analyze and learn from. 

### Case 2: Sandboxing

Because of the ease of changing the configuration of the cluster, we can dive into a bunch of neat ways to use it. Docker networks are isolated, meaning we can sandbox with potentially dangerous binaries and exploits in order to observe its behavior. 

For example, say you have a suspicious binary that you know runs on Linux or a super obfuscated piece of malware in the form of a python or bash script. You can simply copy it over into a container that is on the private network, and run it. Snort and Splunk can be configured to capture network communications and bytes written to memory or disk. In a realistic scenario where you may have to triage malware, producing the actual indications of compromise is vital to running a succesful incident response investigation.

Another example would be if a new exploit has just been released and your organization does not have any specific mitigation against it. An analyst may use a cluster such as this one to reproduce the exploit, analyze the way it works, and create their own mitigation techniques on the fly. Then, they and their organization aren't left wide open, waiting for a bigger vendor to roll out a rule or signature.

## Limitations

With the way containers work, they may only emulate boxes that can run off of the same kernel as the host; meaning, because we are using a Debian-based VM to host our containers, the containers it hosts may only run debian-based services and applications. If you wish to use a different kernel, it calls for a different VM. There are plenty of ways around this issue, but the point of containerizing this lab was to cut down on the computing resources needed to run our services.

Furthermore, in its current state, any data produced does not persist after spinning down or deleting containers. There is a way to configure data to persist, but it does require more storage resources if you wish to do so. So, in running labs, I suggest you collect what you need from your session before spinning the cluster down. 

This lab is subject to grow in the future, enhancing the security of the cluster by integrating a heavier infrastructure that allows us to follow actual best practices. The current state of the lab leaves it very vulnerable, and I do not recommend hosting it in the cloud or exposing it to the internet just yet. 

---
```
TODO:
* Create Installation and Usage Guide
* Fix Splunk time setting during build time

NOTES:
* Docker volumes used to mount drives to the splunk container for log processing.
```
