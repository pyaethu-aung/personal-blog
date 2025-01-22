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
