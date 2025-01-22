+++
title = "Hot Reload in Go"
date = '2025-01-22T03:19:00+07:00'
draft = true
lang = "en"
slug = "hot-reload-in-go"
tags = ["development", "go"]
+++

Many compiled languages like Go don’t natively support hot reloading. Flutter is an exception. For developers, the typical process of writing code and then manually recompiling in the terminal might seem fine at first. However, as the frequency increases, it can become quite frustrating. For people like me, who tend to use a code editor and terminal separately, it’s probably even worse.

The main pain point here is **context switching**. When working on complex code with nested iterations during development or debugging, it becomes challenging to maintain focus.

When you need to compile and test your code inside a Docker container, the process becomes even more cumbersome. Mounting your current codebase as a volume into the container adds extra steps and complexity to the development workflow.

## Air for Hot Reloading
I use [Air](https://github.com/air-verse/air) for live reloading. It’s just that an ex-coworker introduced me to it, and I’ve been using it ever since. While there are some drawbacks, I’m currently using Air for the image processing server I’m working on.

Installing Air is easy. Run this command to install it:
```bash
go install github.com/air-verse/air@latest
```

Set up a configuration file, `.air.toml`, and start your project with this command:
```bash
air -c .air.toml
```

You can initialize the `.air.toml` file automatically with:
```bash
air init
```

## Air in Dockerfile
Since I often run projects with Docker, I install Air directly in the **Dockerfile** and use [Docker Compose](https://docs.docker.com/compose/) to mount the codebase directory for live reloading.

Dockerfile
```dockerfile
FROM golang:1.23.4-bookworm

WORKDIR /app

RUN go install github.com/air-verse/air@latest

COPY . .
RUN go mod download

CMD ["air", "-c", ".air.toml"]
```

docker-compose.yaml
```yaml
services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
    env_file:
      - .env
    ports:
      - 8080:8080
    volumes:
      - ./:/app
```

At the time of writing this post, [the latest release](https://github.com/air-verse/air/releases/tag/v1.61.7) of Air doesn’t support **Go versions below 1.23**. If, for some reason, you’re using an older Go version, you’ll need to install an earlier version of Air that’s compatible.

## Last But Not Least

I’d recommend setting a slightly higher value for the `rerun_delay` in the `.air.toml` file. Without this adjustment, Air might unnecessarily trigger builds immediately after saving code changes.
