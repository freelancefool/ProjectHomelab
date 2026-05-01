# ProjectHomelab
My attempt at learning docker. This will not be anything special.

## Goals
1. Learn the basics of Docker. 
    a. This will be something that comes with using Docker.
    b. Most learning will probably happen as I hit each wall or knowledge gap.
2. Learn how to manage Splunk within a Docker container.
    a. This is the driving force behind learning containers. I will likely need to learn how to manage Splunk in a container as part of a job, so I might as well learn it at home. 
    b. Containers seem interesting. I like the idea of applications running in their own space. 
3. As I understand how to manage Splunk within Docker containers and learn Docker in general, expand this lab.
    a. Splunk
    * Again this is about learning how to manage Splunk in a container as step 1.
    b. Dokuwiki
    * I think it would be fun to throw all of the documentation for this project into a Wiki. Dokuwiki seems easy to manage and maintain.
    c. nginx
    * Once I feel like I have the basics of exposing Docker containers for microservices, I would like to incorporate nginx to handle port forwarding to all of the applications. This will allow the use of DNS names on port 443 for ease of use.
    d. Ansible
    * Once nginx is up and running to provide routing for traffic, things should probably be moved to Ansible for deployment options. This is also a good tool to learn to help enforce settings for Docker and the host running Docker.
    e. Cloudflare + TLS certs
    * Cloudflare has the options and documentation to set up DNS requests to generate TLS certificates through API calls to work with Let's Encrypt. I think this will be an excellent upgrade option to pursue once the lab is able to be deployed quickly with Ansible. This seems like the next step.
    f. Apache
    * It would be pretty nice if I can host apps locally or host the license file locally so that I can have Splunk pull it down. This would add some complexity to the homelab as Apache would need to be running before Splunk could attempt to start. This might move up the priority list. Unsure at this time.
4. You don't know what you don't know. I'm sure that I will add more things to this lab or try to experiment some.

### Docker

Before I can put things into a Docker Compose file, I need to know how to manage the platform on the CLI. This basic command will handle port mapping, accept the license agreements, set a password, define the container name, and what image we are using:

```docker run -d -p 8000:8000 -p 8088:8088 -p 8089:8089 -p 9997:9997 -e "SPLUNK_START_ARGS=--accept-license" -e "SPLUNK_GENERAL_TERMS=--accept-sgt-current-at-splunk-com" -e "SPLUNK_PASSWORD=$password" --name splunk splunk/splunk:latest```

In the future, I will need to either tokenize the SPLUNK_PASSWORD or pass the argument to the Docker Compose statement at startup. Until then, the `$password` placeholder will need to work. I also need to define volumes, Splunk apps to download at startup, and maybe some other variables.

The Docker Compose startup command will look something like this:
```SPLUNKBASE_USERNAME="$splunkbase_username" SPLUNKBASE_PASSWORD="$splunkbase_password" SPLUNK_PASSWORD="$password" docker-compose up -d```
While working on passing a randomized password to docker-compose, I learned that it needs to be in double quotes to work. Probably obvious, but I'm going to point it out.

From the Splunk docs:

To validate HEC is provisioned properly and functional:
```curl -k https://localhost:8088/services/collector/event -H "Authorization: Splunk abcd1234" -d '{"event": "hello world"}' {"text": "Success", "code": 0}```

I haven't tested HEC, yet. I'm sure that this statement will work. I think I would rather see HEC managed from a self-hosted Splunk app over declaring a bunch of HEC tokens at startup with no context for their use, though. I'll work on this part later.