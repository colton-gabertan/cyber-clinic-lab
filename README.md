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

---
```
TODO:
* Create Installation and Usage Guide
  - finish limitation section ^^
* Configure syslog forwarding on Metasploitable2
* Configure snort log forwarding from Snort
* Fix Splunk time setting during build time

NOTES:
* Automation of HTTP Event Collector token working
  - logs stdout as a proof of concept
```
