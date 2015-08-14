---
layout: post
title: "Running Docker in GoCD Agent that runs in a Docker"
date: 2015-08-13 19:08:27 -0700
comments: true
categories: [GoCD, Docker]

---
You may find this article interesting, if you:

- use Docker;
- use GoCD to build docker images;
- run GoCD agent in a docker.

As a base image let's use [Docker-in-Docker](https://hub.docker.com/r/jpetazzo/dind/) image:
```
FROM jpetazzo/dind
```

To run GoCD agent in a docker, we need to run several processes in one container. For this let's use [supervisor](http://docs.docker.com/articles/using_supervisord/):
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

Complete Dockerfile is available at: https://github.com/tispr/docker-gocd/tree/master/gocd-agent-dind/

You can also start playing with existing image by pulling [tispr/gocd-agent-dind](https://hub.docker.com/r/tispr/gocd-agent-dind/):
```
docker pull tispr/gocd-agent-dind
```

## References
- Docker in Docker: https://github.com/jpetazzo/dind/
- Go Continuous Delivery: http://www.go.cd/
- Jenkins DIND: https://github.com/killercentury/docker-jenkins-dind