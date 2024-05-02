---
layout: running
title: Running PostgreSQL in Docker Container
categories: Note
date: 2024-05-02 14:27:13
tags:
  - Docker
  - PostgreSQL
---

Use Postgres locally with Docker<!--more-->

---

![](https://miro.medium.com/v2/resize:fit:4800/format:webp/1*b5UBEuktS0lhRoit39GtGw.jpeg)

# Setup PostgreSQL

Use `docker pull postgres` to Pull [Docker image](https://hub.docker.com/_/postgres)

Or start it with the `run` command

```bash
docker run -e POSTGRES_PASSWORD=mysecretpassword -p 5432:5432 postgres
```

The environment variable `POSTGRES_PASSWORD` set the password for the database

The default port for the database is 5432, use `-p 5432:5432` to exposed it on the host

# Setup pgAdmin

Use `docker pull dpage/pgadmin4` to Pull [Docker image](https://hub.docker.com/r/dpage/pgadmin4/)

Run the pgAdmin by provide environment variables

```bash
docker run --name pgadmin
 -p 15668:80
 -e "PGADMIN_DEFAULT_EMAIL=admin@admin.com"
 -e "PGADMIN_DEFAULT_PASSWORD=mysecretpassword"
 -d dpage/pgadmin4:latest
```

`PGADMIN_DEFAULT_EMAIL` and `PGADMIN_DEFAULT_PASSWORD` for login

`-p 15668:80` to Expose the 80 port

{% asset_img run-pgadmin.png run-pgadmin %}

Find the ip address of PostgreSQL database container that running in docker

```bash
$ docker ps
CONTAINER ID   IMAGE                   COMMAND                  CREATED      STATUS          PORTS                            NAMES
9e6776ff8b38   dpage/pgadmin4:latest   "/entrypoint.sh"         2 days ago   Up 25 minutes   443/tcp, 0.0.0.0:15668->80/tcp   agitated_mayer
393d048a6e9b   postgres                "docker-entrypoint.s…"   3 days ago   Up 35 minutes   0.0.0.0:5432->5432/tcp           vibrant_cartwright
```

```bash
$ docker inspect vibrant_cartwright -f "{{json .NetworkSettings.Networks.bridge.IPAddress }}"
"172.17.0.2"
```

Then use it as host address of pgAdmin server connection

{% asset_img connect-db.png connect-db %}

# Connect to the database in container

## psql console

Check the container name

```bash
$ docker ps
CONTAINER ID   IMAGE      COMMAND                   CREATED      STATUS          PORTS                    NAMES
393d048a6e9b   postgres   "docker-entrypoint.s…"   3 days ago   Up 37 seconds   0.0.0.0:5432->5432/tcp   vibrant_cartwright
```

A `exec` command to open the connection using psql console

```bash
$ docker exec -it vibrant_cartwright psql -U postgres postgres
psql (16.2 (Debian 16.2-1.pgdg120+2))
Type "help" for help.
```

Use `\d` to show database information

```bash
postgres=# \d
Did not find any relations.
```

## Connection string for host application

If you want to connect the database by connection string, for example `.env` file

```
DATABASE_URL=postgres://postgres:mysecretpassword@localhost:5432/postgres
```

---

[Running PostgreSQL in Docker Container with Volume](https://medium.com/@basit26374/how-to-run-postgresql-in-docker-container-with-volume-bound-c141f94e4c5a)
