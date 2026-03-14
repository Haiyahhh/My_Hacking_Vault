---
tags:
  - 🧠
up:
  - "[[02 - Enumeration]]"
  - "[[02 - Privilege Escalation]]"
aliases: []
creation_date: 2026-02-26
last_modified: 2026-02-26
---
# 🧠 [[Docker White-box Challenge Enumeration]]
**Primary:** [[01 - Web Security]], [[01 - Network Security]]

**Secondary:** [[02 - Enumeration]], [[02 - Privilege Escalation]]

## TLDR
Sometimes when solving a Web Security or Network Security challenge, you'll be given the source code of the challenge in the form of a Docker source folder. The challenge would then sometime require you to solve for the flag offline on your own machine first before going for the real one.

This note exist solely for the purpose of giving you the instruction as well as some commands and tips when solving such challenges.

## Enumeration
### Read the Dockerfile
The first thing you need to do is to read the **Dockerfile**, this file contains the instructions that Docker will run to create the image the moment you run `docker run` on the terminal.

**Example Dockerfile (src: [[BKSEC - Ancient Scroll]]):**

```dockerfile
FROM php:7.2-apache

WORKDIR /var/www/html

COPY src/ /var/www/html/

COPY php-src/libphp7_amd64.so /tmp/libphp7_amd64.so
COPY php-src/libphp7_arm64.so /tmp/libphp7_arm64.so

RUN ARCH=$(dpkg --print-architecture) && \
    if [ "$ARCH" = "amd64" ]; then \
        mv /tmp/libphp7_amd64.so /usr/lib/apache2/modules/libphp7.so; \
    elif [ "$ARCH" = "arm64" ]; then \
        mv /tmp/libphp7_arm64.so /usr/lib/apache2/modules/libphp7.so; \
    else \
        echo "Unsupported architecture: $ARCH"; exit 1; \
    fi && \
    rm /tmp/libphp7_*.so

RUN echo "BKSEC{REDACTED}" > /flag.txt
RUN chown -R www-data:www-data /var/www/html && chmod -R 755 /var/www/html
```

Some commands used in a dockerfile.

|Command|<syntax / what after the command>|Description|
|---|---|---|
|`FROM`|`<image>[:<tag>] [AS <name>]`|Specifies the base image for the Docker image (get from https://hub.docker.com/). All subsequent instructions will be built on top of this base image.|
|`WORKDIR`|`<path>`|Sets the working directory for any `RUN`, `CMD`, `ENTRYPOINT`, `COPY`, or `ADD` instructions that follow it in the Dockerfile.|
|`COPY`|`<src>... <dest>`|Copies new files or directories from `<src>` (path on the host that's relative to the directory that's running the `docker run` command) and adds them to the filesystem of the container at the path `<dest>`.|
|`ADD`|`<src>... <dest>`|Can do anything `COPY` can. On top of that, `ADD` can download file on a remote URL and put in the container, and auto-decompress zip files.|
|`RUN`|`<command>` or `["executable", "param1", "param2"]`|Running shell command **at build time (`docker build`)** based on the image you are using (`RUN <command>) or running command with executables and lose all shell features (things like chaining and output redirection)|
|`CMD`|`<command>` or `["executable", "param1", "param2"]`|Running shell command **at build time (`docker run`)** based on the image you are using (`RUN <command>) or running command with executables and lose all shell features (only 1 CMD/Dockerfile)|
|`ENTRYPOINT`|`["executable", "param1", "param2"]` or `command`|Setup the default command that will run when `docker run` was executed and it can not be overridden by running a different command like `docker run /bin/bash`(only 1 ENTRYPOINT/Dockerfile)|
|`EXPOSE`|`<port> [<port>/<protocol>...]`|Informs Docker that the container listens on the specified network ports at runtime. `EXPOSE` does not actually publish the port (meaning it does not really do anything, just a documentation).|
|`ENV`|`<key>=<value> ...`|Sets environment variables. These variables can be accessed by subsequent instructions in the Dockerfile and by the running container.|
|`ARG`|`<name>[=<default value>]`|Defines a variable that users can pass at build-time to the builder with the `docker build --build-arg <varname>=<value>` command.|
|`VOLUME`|`["/data"]` or `/data`|Tells Docker to save the data in this specific folder permanently on your actual computer, so it isn't lost when the container is deleted.|
|`USER`|`<name>[:<group>]` or `<UID>[:<GID>]`|Sets the user name or UID to use when running the image and for any `RUN`, `CMD`, and `ENTRYPOINT` instructions that follow it.|
|`HEALTHCHECK`|`[OPTIONS] CMD <command>` or `NONE`|Tells Docker how to test a container to check if it is still working.|
|`LABEL`|`<key>=<value> [<key>=<value> ...]`|Adds metadata to an image. A `LABEL` is a key-value pair.|
|`SHELL`|`["executable", "parameters"]`|Allows the default shell used for the shell form of `RUN` commands to be overridden.|
|`STOPSIGNAL`|`<signal>`|Sets the system call signal that will be sent to the container to exit.|

### Read the docker-compose.yml
Unlike the Dockerfile, the compose file is the master blueprint for the target infrastucture, when looking for information inside a compose file, for most of the time, you only need to look for some specific configurations: 
1. `environment` or `env_file`: environment variables. (looking for flags, hardcoded credentials, tokens,...)
2. `volumes`: the mount points on the container, dangerous mount points including:
	- `/var/run/docker.sock`: This is the API that controls Docker itself, being able to compromise this folder allows you to create a new docker container that has **the whole host root directory mounted** onto it. 
	- `/`: just as explain above, almost no one does this.
	- `/etc`: this folder contains core configuration files. If you manage to become root of a container that has this folder mounted, imagine what you can do. (e.g. create a new root user)
	- `/proc`: this is the interface of the host's internal data structure. 
	- `/dev`: contains the device files on the host, including the raw hard drives. using tools like `fdisk` or `debugfs` allows you to completely bypass permission and execute whatever you want.
3. `privileged: true`: No one does this, this gives the root of the container the same power as the root on the host.
4. `cap-add`: 
	- `SYS_ADMIN`: pretty much `privileged:true`.
	- `SYS_MODULE`: allow insertion or removal of Linux kernel modules on the hosts' OS. Since the container and the hosts share the same kernel, being able to insert a malicious kernel module can escalate to root immediately.
	- `SYS_PTRACE`: allow the container to use `ptrace()` to trace and debug arbitrary system processes, if `pid: host` is also enabled, `ptrace()` can be used to inject malicious shellcode into the processes that is running as root on the host machine
	- `DAC_OVERRIDE`: allow the read of any files or directory.
	- `DAC_READ_SEARCH`: allow the read and write capability and execute anything.
5. `build`: points to a local directory that contains the custom **Dockerfile**.
6. `ports`: disclose which port on the host is attached to which port on the container.

### Dependency Auditing
After you finish with Dockerfile and the compose file, the next thing is looking for vulnerable library, dependencies that are existing inside the codebase. Checking out files like `requirements.txt`, `package.json`, `pyproject.toml`, `uv.lock`, `pom.xml`, `build.gradle`, ....

After finding out the vulnerable dependencies then we'll note some of the found vulnerabilities to understand what clues to look for inside the codebase.

### Route Mapping
Read all of the codes of all routers in the app in order to understand how each router works, its role and how they interact with one another. Mapp out all of their route.

It's best if you can find a documentation file inside the codebase that list out the route mapping.

**Front-end frameworks**
- **React**: `App.tsx`, `App.jsx`,`routes.ts`
- **Angular**: `app-routing.module.ts`
- **Vue.js**: `router/index.js`, `router.js`.

**Python back-ends**
- **Flask/FastAPI**:`app.py`, `main.py`, `routes.py`, ....
- **Django**: `urls.py`Ru

**Java back-ends**
- **Spring Boot**: `*Controller.java`(e.g. `UserController.java`, ...) and look for keywords like `@RestController`, `@RequestMapping(` or `@GetMapping`.

**Node.js / JavaScript back-ends**
- **Express.js**: `app.js`, `index.js` or a `routes/` directory (look for keywords `app.get()`, `app.post()` or `router.use`)

**PHP frameworks**
- **Lavarel**: `routes/` directory, `web.php` file or `api.php` file.

**Ruby on Rails frameword**
- Look for `config/routes.rb`.

### Logical Vulnerabilities
Now you can look for logical vulnerabilities in the codebase and exploit them.

## Related Usage

### CTFs
[[BKSEC - Ancient Scroll]]
[[BKSEC - Low Effort SNS 2]]

---

**References:** [Link](https://www.google.com/search?q=url)