---
layout: post
title: "Running Docker in GoCD Agent that runs in a Docker"
date: 2015-08-13 19:08:27 -0700
comments: true
categories: [GoCD, Docker, Continuous Delivery]

---
Setting up Continuous Delivery may be challenging. However, it is definitely worth time and resource investment,
if you want to release often, maintain high quality of released products and have full control over the complete process.
[GoCD](http://www.go.cd/) is build on a pipelines concept described in [Continuous Delivery book](http://martinfowler.com/books/continuousDelivery.html).
[Docker](https://www.docker.com/) greatly fits into the concept by solving the problem of packaging applications with all required dependencies and configuration into a standardized container unit.
Common build pipeline will use a Dockerfile as an input, create image and publish it to a Docker registry.
Common deploy pipeline will use a Docker registry image as an input and deploy it to proper environment when is updated.

GoCD in its basic setup is distributed and has Server and one or more Agents, which are running actual Jobs and Tasks.
Server can be customized by adding new plugins, or by setting up a default configuration file with backed up pipelines.
Agents may have different resources to run different types of jobs, e.g. separate Agents to build java and node.js projects.
Same as for any other software deliverable, packaging GoCD Server and Agent into Docker containers simplifies their deployment and maintainability.
One of the challenges in this setup will be running your existing Docker pipelines in an Agent, which is running in a Docker container.

So, in brief, you may find this article interesting, if you:

- use GoCD;
- use Docker;
- use GoCD to build Docker images;
- run GoCD agent in a Docker.

The most important part of this setup is a [Docker-in-Docker](https://hub.docker.com/r/jpetazzo/dind/), which we use as a base image:
```
FROM jpetazzo/dind
```

To run GoCD agent in a Docker, we need to run several processes in one container. For this let's use [supervisor](http://docs.docker.com/articles/using_supervisord/):
```
RUN apt-get -y install supervisor
ADD supervisord.conf /etc/supervisor/conf.d/supervisord.conf
CMD ["/usr/bin/supervisord"]
```

Configuration file for supervisor will contain section per each process, including supervisor itself:

```
[supervisord]
nodaemon=true

[program:docker]
priority=10
command=wrapdocker

[program:gocdagent]
priority=20
command=/bin/bash -c "/etc/init.d/go-agent start"
```

Some additional tweaks may be required to run GoCD agent properly. One of them is setting `GO_SERVER` in agent configuration. Let's assume that it is available as environment variable with the same name:
```
echo export GO_SERVER=$GO_SERVER >> /etc/default/go-agent
```

Another update will allow to run `sudo docker` without interactive password prompt:
```
echo go ALL=NOPASSWD: /usr/bin/docker >> /etc/sudoers
```

And complete GoCD command will look like this:
```
command=/bin/bash -c "echo export GO_SERVER=$GO_SERVER >> /etc/default/go-agent && echo go ALL=NOPASSWD: /usr/bin/docker >> /etc/sudoers && /etc/init.d/go-agent start"
```

You can check complete [Dockerfile](https://github.com/tispr/docker-gocd/tree/master/gocd-agent-dind/) and start playing
with existing image by pulling [tispr/gocd-agent-dind](https://hub.docker.com/r/tispr/gocd-agent-dind/):
```
docker pull tispr/gocd-agent-dind
```

Now nothing stops you from building GoCD pipeline GoCD Server and GoCD Agents Docker containers.

## References
- Docker in Docker: https://github.com/jpetazzo/dind/
- Go Continuous Delivery: http://www.go.cd/
- Jenkins DIND: https://github.com/killercentury/docker-jenkins-dind