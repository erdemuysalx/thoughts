---
title: Linux Process Isolation and Docker Containers
description: Introduction to Containers
date: '2021-09-18T17:41:18.217Z'
categories: []
keywords: []
slug: /@erdemuysalx/linux-process-isolation-and-docker-containers-1d134ebb796c
---

![](https://cdn-images-1.medium.com/max/800/1*ekWrA5PWqYMHgc_82A5BNQ.png)

#### **Containers**

Containers are an abstraction at the app layer that packages code and dependencies together. Multiple containers can run on the same machine and share the OS kernel with other containers, each running as isolated processes in userspace. In the end, the container is a process that runs on the OS kernel but with a little extra isolation.

#### **Virtual Machines**

Virtual Machines (VMs) are an abstraction of physical hardware turning one server into many servers, the hypervisor allows multiple VMs to run on a single machine. Each VM includes a full copy of an operating system, the application, necessary binaries, and libraries - taking up tens of GBs. VMs can also be slow to boot.

![](https://cdn-images-1.medium.com/max/800/0*VgBN-MJj_eXYjEtY.png)

#### Why Containers?

*   Isolation: Containers are independent of the OS behind the container.
*   Repeatability: Containers will run exactly the same in another machine since everything your app needs is packaged within the container.
*   Security: A container can not access either processes or containers on the OS, each one has an individual process namespace.
*   Clean setup/remove: Set up or remove the whole app and its dependencies with only one command.

#### Advantages of Containers

*   Run in seconds instead of minutes.
*   Consume fewer resources results in less disk space.
*   Uses less memory.
*   Does not need a full OS.
*   Provide faster deployment.
*   Quick testing.

#### How do Containers work?

Actually, there is no such concept called as the container in Linux. We can verify this by grepping the source code of Linux. But then how do these containers work?

We as developers, use interfaces called container runtimes such as Docker in order to create what we call containers in a more user-friendly way. These container runtimes are just programs that make system calls to the OS kernel. Therefore Docker is a program that makes system calls to the Linux kernel for creating containers easily as we desire. Docker is not the main thing behind the containers, it is just an interface that presents our containers in a language that we can understand. So how the containers are implemented in the Linux kernel?

#### **1\. Isolation**

Restricting what the process sees.

**chroot:** The chroot is an operation on Unix-based operating systems that changes the current root directory, as its name refers to it (change root), for the selected running process and children of that process. It is the oldest restriction/isolation operation in Unix-based operating systems among the other utility operations. It basically creates an illusionistic root directory that isolates the specific command i.e. process from the other processes. This illusionistic fake root directory is often called ‘_chroot jail_’.

**unshare:** The unshare is an operation which is a part of the Linux kernel since 2002 that helps us create namespaces. Just like the chroot operation, unshare operation also provides isolation between processes. There are different types of namespaces as following:

*   User Namespace
*   Process ID (PID) Namespace
*   Network Namespace
*   Mount Namespace
*   Interprocess Communication (IPC) Namespace

Each namespace type has its own unique properties that differ from others.

#### 2\. Resource Controls

Restricting the resources used by the process.

**cgroups:** The cgroups, also known as change groups, is a Linux kernel operation that allows creating a hierarchy for a selected set of processes. The key feature of cgroups operation that distinguishes it from the others is hierarchical management and allocation of system resources like CPU, memory, and storage for the selected process instances of the groups.

### What is Docker?

*   Docker is a tool for running applications in an isolated environment.
*   It is similar to a virtual machine.
*   The app runs in the same environment regardless of which machine it is since it is isolated.
*   The app is just working at any machine regardless of which OS it uses, what type of hardware is equipped with etc.
*   Docker is a standard for software deployment nowadays.

![](https://cdn-images-1.medium.com/max/800/1*pebo5ZC35HDY6osrty5F8A.png)

#### Install Docker

Since there are a bunch of sources about this on the internet, I will not cover it in this article. I recommend you to follow the official Docker page for the installation.

[https://docs.docker.com/get-docker/](https://docs.docker.com/get-docker/)

#### Docker Image

Image is a template for creating an environment of your choice (DB, web app, or others). It is a snapshot of your application that has everything need to run it.

#### Container

Running instances of images called containers.

#### Pulling an Image

```
docker pull nginx  
docker images  # list the images on your machine  
docker image ls  # list the images on your machine
```

#### Running a Container

```
docker run nginx:latest  # run the container  
docker run — name given\_name nginx:latest  #give a name to container  
docker container ls or docker ps  # list the running containers
```

#### Exposing Ports

Since containers work in an isolated environment, most of the time, you might want to expose some ports in order to view or interact with your applications.

`-p` flag manages the port relations of your Docker container. For example, the following command line exposes port 80 of the Docker container and maps it to localhost’s 8080 port:

`docker run -d -p 8080:80 nginx:latest`

We can also expose multiple ports for a single Docker container using more than one `-p` flag. See the following command line:

`docker run -d -p 3000:80 -p 8080:80 nginx:latest`

#### Managing Containers

```
docker ps  # list the running container  
docker ps -a  # list all containers  
docker ps -q  # list the containers with only numeric ids  
docker stop container\_id  # stops the container with indicated id  
docker start container\_id  # starts the container with indicated id  
docker rm container\_id  # delete the container with indicated id
```

#### Volumes

Volumes are the utility functions for Docker containers that allow sharing of data, files & folders, between host and container or between containers.

**Volumes Between Host and Container**

docker -run — name cwebsite -v /Users/user/Desktop/website:/usr/share/nginx/html:ro -d -p 8080:80 nginx:latest

*   **ro:** read-only (container only allowed to read file and folders)
*   **wo:** write-only (container only allowed to write file and folders)

If you don't specify it will allow both write and read. If we create a folder or a file in the host that will appear in the container as well.

**Volumes Between Containers**

`docker run --name website-copy --volumes-from website -d -p 8081:80 nginx:latest`

#### Docker Specific Files

**Dockerfile:** A Dockerfile is a kind of manifest that describes what should that docker image contain, and how it should be constructed.

*   Allow us to build our own images.
*   Series of steps that define your Docker image.
*   Dockerfile keywords: FROM, WORKDIR, ADD, RUN, CMD, etc.

To create an image with a Dockerfile we simply run this command:

`docker build -t tag:version path\_of\_dockerfile`

**.dockerignore:** You have designed an app and coded it with your favorite programming language. Now you want to deploy it using Docker containerization technology. At the same time, you pushed your project to GitHub, thus your project folder contains some .git folders that are required for the GitHub repository. These types of folders or files are unnecessary for the deployment process. That’s why we do not want to include them in our Docker image. For this sake, we simply add the names of folders and files that we do not want to include into our Docker image to the .dockerignore file.

#### Build an Image for Your Python App

Let’s say you want to have a web app written with Python programming language using the Flask framework.

```
cd /path/to/python-docker
pip3 install Flask
pip3 freeze > requirements.txt$ touch app.py
```

After following the steps above, you have created a Python file and you are ready to write your code. In this article, for the sake of simplicity, we are going to write a few lines of code that make a simple web request, a Hello World web app. Now you can open your project file with your favorite IDE or text editor selection like VSCode or Sublime, etc., and put the code below inside the file.

```
from flask import Flask
app = Flask(__name__)

@app.route('/')
def hello_world():
    return 'Hello, Docker!'
```

You have tested the app by typing the run command `python3 -m flask run` . When you run this command it listed that the app is running on your local machine with a port number 5000. You tested and confirmed that the app is working properly as you desired by opening a new browser and navigate to `[http://localhost:5000](http://localhost:5000.)`[.](http://localhost:5000.)

Now you want to launch and serve this web app to users around the world. The traditional way of doing this is serving your app on a server like Amazon, Google, etc. Using this way is working, but it is not efficient because when you need updates or bug fixes you need to connect to the server for example via SSH and do what you need just like on your local machine which causes interruption on your served app.

Therefore you decided to deploy your app using much more modern technology i.e. Docker. Of course, there are more advanced ways of deploying apps using different stacks of technologies like Docker + Kubernetes + Jenkins CI/CD tools, etc. but these are not within the scope of this article and will not be covered here. Now your deployment technology is Docker and for deploying your web app using Docker, you will eventually need a Dockerfile which we discussed in earlier titles.

```
FROM python:3.8-slim-buster
WORKDIR /app
COPY requirements.txt requirements.txt
RUN pip3 install -r requirements.txt
COPY . .
CMD [ "python3", "-m" , "flask", "run", "--host=0.0.0.0"]
```

You can now build your web app’s Docker image using the Dockerfile above and by typing the command `docker build -t python-docket:v0` . Afterward, you achieved a Docker image that wraps your web app and it is ready to deploy.

Thank you for reading this article, and I will see you in the next one!

#### Resources

*   [https://man7.org/linux/man-pages/man2/chroot.2.html](https://man7.org/linux/man-pages/man2/chroot.2.html)
*   [https://www.ibm.com/docs/en/zos/2.2.0?topic=descriptions-chroot-change-root-directory-execution-command](https://www.ibm.com/docs/en/zos/2.2.0?topic=descriptions-chroot-change-root-directory-execution-command)
*   [https://man7.org/linux/man-pages/man1/unshare.1.html](https://man7.org/linux/man-pages/man1/unshare.1.html)
*   [https://www.nginx.com/blog/what-are-namespaces-cgroups-how-do-they-work/](https://www.nginx.com/blog/what-are-namespaces-cgroups-how-do-they-work/)
*   [https://man7.org/linux/man-pages/man7/cgroups.7.html](https://man7.org/linux/man-pages/man7/cgroups.7.html)
*   [https://www.ibm.com/docs/en/spectrum-symphony/7.3.0?topic=limits-control-groups-cgroups-limiting-resource-usage-linux](https://www.ibm.com/docs/en/spectrum-symphony/7.3.0?topic=limits-control-groups-cgroups-limiting-resource-usage-linux)
*   [https://docs.docker.com/get-started/overview/](https://docs.docker.com/get-started/overview/)
*   [https://docs.docker.com/engine/reference/builder/](https://docs.docker.com/engine/reference/builder/)
*   [https://docs.docker.com/language/python/build-images/](https://docs.docker.com/language/python/build-images/)