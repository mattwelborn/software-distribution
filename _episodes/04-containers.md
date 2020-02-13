---
title: "Containers"
teaching: 0
exercises: 0
questions:
- "Key question (FIXME)"
objectives:
- "First learning objective. (FIXME)"
keypoints:
- "First key point. Brief Answer to questions. (FIXME)"
---
## Get docker

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

The output explains what happened.

You can list the images that you have locally, and see that the hello-world image was downloaded.

~~~
docker images
~~~
{: .language-bash}


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

{% include links.md %}

