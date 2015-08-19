---
layout: post
title: "Running Docker in a GoCD Agent that runs in a Docker"
date: 2015-08-18 19:08:27 -0700
comments: true
categories: [GoCD, Docker, Continuous Delivery]

---
Setting up Continuous Delivery may be challenging. However, it is definitely worth the time and resource investment,
if you want to release often, maintain a high quality of released products and have full control over the complete process.
[GoCD](http://www.go.cd/) is built on a pipelines concept described in [Continuous Delivery book](http://martinfowler.com/books/continuousDelivery.html).
[Docker](https://www.docker.com/) greatly fits into the concept, because it solves the problem of packaging applications with all required dependencies and configurations into a standardized container unit.
A common build pipeline will use a Dockerfile as an input, create an image and publish it to a Docker registry.
A common deploy pipeline will use a Docker registry image as an input and deploy it to a proper environment when it is updated.

GoCD in its basic setup is distributed and has a Server and one or more Agents, which are running actual Jobs and Tasks.
The Server can be customized by adding new plugins, or by setting up a backed-up pipeline configuration.
The Agents may have different resources to run different types of jobs, i.e. separate Agents to build java and node.js projects.
Same as for any other software deliverable, packaging GoCD Server and Agents into Docker containers simplifies their deployment and maintainability.
One of the challenges in this setup is running your existing Docker pipelines in an Agent, which is running in a Docker container.

So, in brief, you may find this article interesting, if you:

- use GoCD;
- use Docker;
- use GoCD to build Docker images;
- run GoCD agents in a Docker.

The most important part of this setup is a [Docker-in-Docker](https://hub.docker.com/r/jpetazzo/dind/), which we use as a base image:
```
FROM jpetazzo/dind
```

To run a GoCD agent in a Docker, we need to run several processes in one container. For this let's use [supervisor](http://docs.docker.com/articles/using_supervisord/):
```
RUN apt-get -y install supervisor
ADD supervisord.conf /etc/supervisor/conf.d/supervisord.conf
CMD ["/usr/bin/supervisord"]
```

The configuration file for the supervisor will contain a section per each process, including the supervisor itself:

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

Some additional tweaks may be required to run your GoCD agent properly. One of them is setting `GO_SERVER` in your agent configuration. Let's assume that it is available as an environment variable with the same name:
```
echo export GO_SERVER=$GO_SERVER >> /etc/default/go-agent
```

Another update will allow to run `sudo docker` without an interactive password prompt:
```
echo go ALL=NOPASSWD: /usr/bin/docker >> /etc/sudoers
```

And the complete GoCD command will look like this:
```
command=/bin/bash -c "echo export GO_SERVER=$GO_SERVER >> /etc/default/go-agent && echo go ALL=NOPASSWD: /usr/bin/docker >> /etc/sudoers && /etc/init.d/go-agent start"
```

You can check the complete [Dockerfile](https://github.com/tispr/docker-gocd/tree/master/gocd-agent-dind/) and start playing
with existing image by pulling [tispr/gocd-agent-dind](https://hub.docker.com/r/tispr/gocd-agent-dind/):
```
docker pull tispr/gocd-agent-dind
```

Now nothing stops you from setting up a GoCD pipeline to manage dockerized GoCD Server and GoCD Agents.

## References
- Docker in Docker: https://github.com/jpetazzo/dind/
- Go Continuous Delivery: http://www.go.cd/
- Jenkins DIND: https://github.com/killercentury/docker-jenkins-dind
