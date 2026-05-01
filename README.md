# ProjectHomelab
My attempt at learning docker. This will not be anything special.

## Goals
1. Learn the basics of Docker. 
    1. This will be something that comes with using Docker.
    2. Most learning will probably happen as I hit each wall or knowledge gap.
2. Learn how to manage Splunk within a Docker container.
    1. This is the driving force behind learning containers. I will likely need to learn how to manage Splunk in a container as part of a job, so I might as well learn it at home. 
    2. Containers seem interesting. I like the idea of applications running in their own space. 
3. As I understand how to manage Splunk within Docker containers and learn Docker in general, expand this lab.
    1. Splunk
        * Again this is about learning how to manage Splunk in a container as step 1.
    2. Dokuwiki
        * I think it would be fun to throw all of the documentation for this project into a Wiki. Dokuwiki seems easy to manage and maintain.
    3. nginx
        * Once I feel like I have the basics of exposing Docker containers for microservices, I would like to incorporate nginx to handle port forwarding to all of the applications. This will allow the use of DNS names on port 443 for ease of use.
    4. Ansible
        * Once nginx is up and running to provide routing for traffic, things should probably be moved to Ansible for deployment options. This is also a good tool to learn to help enforce settings for Docker and the host running Docker.
    5. Cloudflare + TLS certs
        * Cloudflare has the options and documentation to set up DNS requests to generate TLS certificates through API calls to work with Let's Encrypt. I think this will be an excellent upgrade option to pursue once the lab is able to be deployed quickly with Ansible. This seems like the next step.
    6. Apache
        * It would be pretty nice if I can host apps locally or host the license file locally so that I can have Splunk pull it down. This would add some complexity to the homelab as Apache would need to be running before Splunk could attempt to start. This might move up the priority list. Unsure at this time.
4. You don't know what you don't know. I'm sure that I will add more things to this lab or try to experiment some.
    1. I'll probably get better with version control as part of this project. Currently, I've learned I need to make protections so I can't merge straight to main, because I will.
    2. I'm going to get better with markdown as I document this in Git. The world can laugh at my mistakes with me.

### Splunk

Before I can put things into a Docker Compose file, I need to know how to manage the platform on the CLI. This basic command will handle port mapping, accept the license agreements, set a password, define the container name, and what image we are using:

```docker run -d -p 8000:8000 -p 8088:8088 -p 8089:8089 -p 9997:9997 -e "SPLUNK_START_ARGS=--accept-license" -e "SPLUNK_GENERAL_TERMS=--accept-sgt-current-at-splunk-com" -e "SPLUNK_PASSWORD=$password" --name splunk splunk/splunk:latest```

In the future, I will need to either tokenize the SPLUNK_PASSWORD or pass the argument to the Docker Compose statement at startup. Until then, the `$password` placeholder will need to work. I also need to define volumes, Splunk apps to download at startup, and maybe some other variables.

The Docker Compose startup command will look something like this:

```SPLUNKBASE_USERNAME="$splunkbase_username" SPLUNKBASE_PASSWORD="$splunkbase_password" SPLUNK_PASSWORD="$password" docker-compose up -d```

While working on passing a randomized password to docker-compose, I learned that it needs to be in double quotes to work. Probably obvious, but I'm going to point it out. I also learned that `docker-compose` is specific to WSL, which is where I wrote this originally. It works fine on RHEL 10 without the hyphen.

From the Splunk docs:

To validate HEC is provisioned properly and functional:
```curl -k https://localhost:8088/services/collector/event -H "Authorization: Splunk abcd1234" -d '{"event": "hello world"}' {"text": "Success", "code": 0}```

#### To Do

I haven't tested HEC, yet. I'm sure that this statement will work. I think I would rather see HEC managed from a self-hosted Splunk app over declaring a bunch of HEC tokens at startup with no context for their use, though. I'll work on this part later.

Add persistent storage for data and configuration. I would like to break things up so the container is ephemeral. `/opt/splunk/etc` and `/opt/splunk/var` should be persistent, I think? I need to read the docs and see if this remains true. I'm used to managing Splunk on dedicated hosts and am behind on containerized Splunk management.

### dokuwiki

Before I can put things into a Docker Compose file, I need to know how to manage the platform on the CLI. 

Lifted from the dokuwiki docs as a starting point:

```docker run -p 8080:8080 --user 1000:1000 -v /path/to/storage:/storage dokuwiki/dokuwiki:stable```

#### To Do


## Requirements

Some of this will be hard. This test box has been in use for a while. Hopefully, whoever reads this has some understanding of Linux in general and the ability to read between the lines. If/when my test host has issues, I will document the environment.

### Ports

To make it easy to track the ports in use with the homelab, have a table:

| Port  | Service   | Context         |
| ----- | --------- | --------------- |
| 8000  | Splunk    | Web UI          |
| 8080  | dokuwiki  | Web UI          |
| 8088  | Splunk    | HEC listener    |
| 8089  | Splunk    | Splunk API      |
| 9997  | Splunk    | Splunk receiver |

### Packages

Obviously, Docker is a must. I am using docker-ce in my homelab and Docker Desktop on my Windows desktop. 

I also use VS Code over ssh to a Linux host or with WSL locally. 

I also have git installed on my Linux host while I have Git Bash installed on my Windows host.

### Admin Rights

Package management will be necessary at a minimum. `root` will probably make things easier. Splunk does have some requirements for changes to the host before the container will work properly. If `firewalld` is in use or `selinux`, rights for these will be required to allow services to operate externally. The user account I use is in the docker group to allow non-privileged users to launch containers.

### Splunk Specific Information

I am using a 6-month developer license with Splunk. This unlocks some features that the free license does not allow. Mostly around using a deployment server. This functionality is something I want to play with. It is not a requirement to start using Splunk. I am also using a Splunk account to pull from splunkbase.splunk.com. This is not a requirement either for someone else to use this project. Just comment out the lines in `docker-compose.yaml`.
