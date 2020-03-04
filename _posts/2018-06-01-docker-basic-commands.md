---
layout: post
title: "Docker 常用命令"
date: 2018-06-01 00:00:00
categories: docker
comments: true
---

```bash
  docker --version
  docker info

  # run container by image
  docker run <repository>
  docker run -p 4000:80 <repository>
  docker run -d -p 4000:80 <repository>

  docker image ls

  docker container ls
  docker container ls --all

  docker container stop <container id>

  docker exec -it <container id> /bin/bash
```

## Dockerfile

```bash
  # Use an official Python runtime as a parent image
  FROM python:2.7-slim

  # Set the working directory to /app
  WORKDIR /app

  # Copy the current directory contents into the container at /app
  ADD . /app

  # Install any needed packages specified in requirements.txt
  RUN pip install --trusted-host pypi.python.org -r requirements.txt

  # Make port 80 available to the world outside this container
  EXPOSE 80

  # Define environment variable
  ENV NAME World

  # Run app.py when the container launches
  CMD ["python", "app.py"]
```

```bash
  # build image
  docker build -t <image name> .
```

```bash
  docker login

  # tag the image
  docker tag <image name> <username>/<repository>:<tag>
  # example
  docker tag friendlyhello john/get-started:part2

  docker push <username>/<repository>:<tag>

  docker run -p 4000:80 <username>/<repository>:<tag>
```

```bash
  docker-compose up -d
  docker-compose build
  dokcer-compose pull

  docker-compose up --build -d
```
