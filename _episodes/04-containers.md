---
title: "Containers"
teaching: 0
exercises: 0
questions:
- "How can I deploy my code in a cross-platform way?"
objectives:
- "Understand what a container is, and why you might use one."
- "Learn basic Docker commands."
- "See examples of running containers with Docker."
- "See examples of building and distributing containers with Docker."
- "Learn how you can run containers in HPC environments."
keypoints:
- "Containers allow code to be packaged in a cross-platform way."
- "Docker provides a build toolchain and runtime environment for containers."
- "Many other runtime environments exist (e.g. Singularity, Shifter, Charlie Cloud, but all of them are compatible with Docker images."
---

# What is a container?
TODO

<img src="../fig/container_evolution.svg" width="800px">
[Image credit](https://kubernetes.io/docs/concepts/overview/what-is-kubernetes/)


> ## Why are containers so popular?
> Containers simplify the interaction between software providers and compute platform providers:
> * For software providers, all platforms look the same.
> * For platform providers, all software looks the same.
{: .callout}


## Terminology

* __Image__: The basis of containers, analogous to a compiled binary in traditional software deployment. An image does not have state and cannot be changed.
* __Container__: A running image.
* __Engine__: The software that executes containers and connects them to the hardware.
* __Registry__: Stores, distributes, and manages images.

## Advantages of containers

* __Platform independence__: Containers run the same across different computational platforms, from your laptop, to HPC, to the cloud.
* __Dependencies__: You get complete control over what software is installed on the system.
* __Reproducibility__: Images are versioned and archived. At any time, you can run an old version of your code, without figuring out how to compile it again.
* __Resource isolation__: Containers have well defined and strongly enforced resource (CPU, memory, network, disk) limits.
* __Resource utilization__: Because resource limits are well defined and strongly enforced, containers can be efficiently scheduled on cloud and HPC resources.

> ## Learn more about containers
> If you want to know more about virtualization, containers and Dockers, freeCodeCamp
> provides an [excellent overview](https://www.freecodecamp.org/news/a-beginner-friendly-introduction-to-containers-vms-and-docker-79a9e3e119b/).
{: .callout}

# A brief tutorial

This section briefly highlights the use of containers with [Docker](https://docker.com).
While there are many container engines, Docker is the industry standard;
containers built with Docker can be run with other engines, such as [Singularity](https://sylabs.io/docs/#singularity), [Shifter](https://docs.nersc.gov/programming/shifter/overview/), and [CharlieCloud](https://hpc.github.io/charliecloud/).
## Get docker

To start, you will need a Docker engine. Follow the links below to download one.

Windows and Mac:

* [Docker Desktop](https://www.docker.com/products/docker-desktop)

Linux:

* [CentOS](https://docs.docker.com/install/linux/docker-ce/centos/)
* [Debian](https://docs.docker.com/install/linux/docker-ce/debian/)
* [Fedora](https://docs.docker.com/install/linux/docker-ce/fedora/)
* [Ubuntu](https://docs.docker.com/install/linux/docker-ce/ubuntu/)

## Hello world

Docker hosts a demonstration "hello world" image. To run it:

~~~
docker run hello-world
~~~
{: .language-bash}

When you run this command, Docker finds an image named `hello-world` on the Docker Hub, the Docker container registry.
The image is then downloaded and run on your computer using the Docker image.
The output further explains what happened:

~~~
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
1b930d010525: Pull complete
Digest: sha256:9572f7cdcee8591948c2963463447a53466950b3fc15a247fcad1917ca215a2f
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
~~~
{: .output}

You can list the images that you have locally, and see that the `hello-world` image was downloaded.

~~~
docker images
~~~
{: .language-bash}

~~~
REPOSITORY                                     TAG                 IMAGE ID            CREATED             SIZE
hello-world                                    latest              fce289e99eb9        13 months ago       1.84kB
~~~
{: .output}


## Build our own hello world

Docker images are build from `Dockerfile`, which live in "build directories". Each image has its own build directory. Create a new directory with the following `Dockerfile`:

~~~
FROM alpine:3.2
CMD ["echo", "Hello world!"]
~~~
{: .language-docker}

Build:

~~~
docker build -t my-hello-world .
~~~
{: .language-bash}

List images:

~~~
docker images
~~~
{: .language-bash}

Run:

~~~
docker run my-hello-world
~~~
{: .language-bash}

Let's publish this landmark achievement, so others can run it. First, we need to rename it to match my DockerHub username.

~~~
docker build -t mattgwelborn/my-hello-world .
~~~
{: .language-bash}

Notice that the build output is different. This time, the build used a cache from the previous build.

Push to DockerHub:

~~~
docker push mattgwelborn/my-hello-world
~~~
{: .language-bash}

Go to hub.docker.com to see the image.

Let's see how someone else might pull down the image from Dockerhub. First, delete the image from our computer:

~~~
docker container prune
docker image rm mattgwelborn/my-hello-world
docker image rm my-hello-world
docker images
~~~
{: .language-bash}


Now try to run:

~~~
docker run mattgwelborn/my-hello-world
~~~
{: .language-bash}

## Compiled code

Compiled code is built inside the `Dockerfile`. Create a new folder with the code `hello.c`:

~~~
#include <stdio.h>

int main () {
    printf("Hello world!\n");
    return 0;
}
~~~
{: .language-c}

and the `Dockerfile`:

~~~
FROM alpine:3.2
RUN apk update && apk add build-base  # install compilers
RUN mkdir /hello
COPY hello.c /hello/hello.c # copy source from host machine into build container
RUN cd /hello && gcc hello.c -o hello
CMD ["/hello/hello"]
~~~
{: .language-docker}

Build:

~~~
docker build -t compiled-hello-world .
~~~
{: .language-bash}

Run:

~~~
docker run compiled-hello-world
~~~
{: .language-bash}

We can also poke around inside an image. Start a container in interactive mode:

~~~
docker run -it compiled-hello-world /bin/sh
# ls
# cd hello
# cat hello.c
# ./hello
~~~
{: .language-bash}

## Python code

Python code can be installed inside the `Dockerfile` using python package management tools. In this case, we'll use conda to install `qcengine` and `psi4`. `psi4` is a quantum chemistry program. `qcengine` is "a quantum chemistry program executor and I/O standardizer". That is, it allows you to run many different QC programs with a common I/O format (QCSchema).

A `Dockerfile` for conda is a little bit involved:

~~~
FROM continuumio/miniconda3
SHELL ["/bin/bash", "-c"]
RUN conda create -n qcengine -c conda-forge -c psi4 qcengine psi4
RUN groupadd -g 999 qcengine && \
    useradd -m -r -u 999 -g qcengine qcengine
USER qcengine
ENV PATH /opt/conda/envs/qcengine/bin/:$PATH
RUN echo "source activate qcengine" > ~/.bashrc
ENTRYPOINT ["qcengine"]
~~~
{: .language-docker}


Build:

~~~
docker build -t qcengine .
~~~
{: .language-bash}

Now we can run `qcengine`:

~~~
docker run qcengine
docker run qcengine info
~~~
{: .language-bash}

Run psi4 with input schema:

~~~
docker run qcengine run psi4 '{"id": null, "schema_name": "qcschema_input", "schema_version": 1, "molecule": {"schema_name": "qcschema_molecule", "schema_version": 2, "validated": true, "symbols": ["O", "H", "H"], "geometry": [0.0, 0.0, -0.24377467, 0.0, -2.82325083, 1.94074873, 0.0, 2.82325083, 1.94074873], "name": "H2O", "identifiers": null, "comment": null, "molecular_charge": 0.0, "molecular_multiplicity": 1, "masses": [15.99491461957, 1.00782503223, 1.00782503223], "real": [true, true, true], "atom_labels": ["", "", ""], "atomic_numbers": [8, 1, 1], "mass_numbers": [16, 1, 1], "connectivity": null, "fragments": [[0, 1, 2]], "fragment_charges": [0.0], "fragment_multiplicities": [1], "fix_com": false, "fix_orientation": false, "fix_symmetry": null, "provenance": {"creator": "QCElemental", "version": "v0.12.0", "routine": "qcelemental.molparse.from_schema"}, "id": null, "extras": null}, "driver": "energy", "model": {"method": "B3LYP", "basis": "Def2-SVP"}, "keywords": {}, "protocols": {}, "extras": {}, "provenance": {"creator": "QCElemental", "version": "v0.12.0", "routine": "qcelemental.models.results"}}'
~~~
{: .language-bash}

You can use a [JSON viewer](http://jsonviewer.stack.hu/) to understand these JSON blobs.

## Building images in practice

We saw one solution for getting your software into an image above: use a package manager. Just like we used `conda` to build an imag for Python-based software, `spack` and EasyBuild may be used to build images for C/C++/Fortran software.

# Containers in HPC

{% include links.md %}

