---
title: 'Trying New Things: Elastic Stack and RockNSM'
date: 2020-03-07 22:25
categories:
  - blog
tags:
  - elastic
  - rockNSM
  - rambling
  - Threat Hunting
published: false
---

## Elastic Stack on *Docker*

Tools I used:
> Fresh Ubuntu 19.10 (just a generic VM copy ready to fly)
> docker-compose.yml from the elastic docks
> Busted ass ancient Dell server running HyperV

had to run `sudo sysctl -w vm.max_map_count=262144`

Either log in or su to your docker user:

```powershell
sudo groupadd docker
sudo usermod -aG docker $USER
newgrp docker
```

My first docker-compose was littered with errors, didn't bother troubleshooting.  Killed everything with fire, had to killall dockerd then ran it again using:

`docker-compose up --build --force-recreate --remove-orphans`

I'm dumber than a bag of hair when it comes to docker, so take it with a grain of salt.  I got this far using `--help` and the docker docs.