# Central Authentication Service (CAS) [![License](https://img.shields.io/hexpm/l/plug.svg)](https://github.com/Jasig/cas/blob/master/LICENSE)

## Introduction

This is a fork of the Docker based distribution of CAS for use at Clluc.

Why do we need it? Because the original one doesn't give us out of the box everything we need to have a CAS instance
for development. At least a self signed certificated has to be added to the keystore in order for the CAS
instance to bootstrap in test mode (with default static users available). So for the sake of convenience we have
forked the original repo and created a more ready to use version for us.

Instructions about how to use it are given in the following sections. For further information about the rest you can
always refer to the [original repo README file](https://github.com/apereo/cas-webapp-docker).

## Starting a CAS instance in a breathe

This repo ships a keystore with an already included self signed certificate that will be used to enable access to
the CAS login page through HTTPS. By default CAS can only be accessed through HTTPS for obvious reasons. In case we
want to use HTTP for authentication we will need to modify the Docker image ourselves.

That have implications when using this repo to run a CAS Docker image for local development. Given that HTTPS is the
only way to access CAS login page, the corresponding redirect URL after a valid login will be forced to be HTTPS as
well, even if we specify an HTTP one. In case we are running our Stockmind API locally over plain HTTP we will need
some mean to redirect to the HTTP counterpart.

That said, let's now explain the steps to run it.

### Building the Docker image

This repo provides with shell scripts that makes things a lot easier for us. Before running the CAS Docker image on
any given machine for the first time we need to build the Docker image. Given that the build process require several
steps it is better to do it as an isolated task separated from running a container from the image.

Creating the Docker image is as easy as running the following command:

```./build.sh $CasVersion```

CasVersion is 5.1.6 at the time of this writing. You can check the latest one at [Dockerhub](https://hub.docker.com/r/apereo/cas/tags/).

### Running the Docker image

After building the image, running it is as simple as this:

```./run.sh $CasVersion```

It will take several seconds for the container to start. Once started, we can access the running CAS instance [locally
via port 8443 through HTTPS](https://localhost:8443/cas).
