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

First of all, a keystore with a self signed certificate has to be generated. It will be used to enable access to
the CAS login page through HTTPS. By default CAS can only be accessed through HTTPS for obvious reasons. In case we
want to use HTTP for authentication we will need to modify the Docker image ourselves.

That have implications when using this repo to run a CAS Docker image for local development. Given that HTTPS is the
only way to access CAS login page, the corresponding redirect URL after a valid login will be forced to be HTTPS as
well, even if we specify an HTTP one. In case we are running our Stockmind API locally over plain HTTP we will need
some mean to redirect to the HTTP counterpart.

That said, let's now explain the steps to run it.

### Create a new certificate

```openssl req -x509 -nodes -config configuracion.cnf -newkey rsa:2048 -sha256 -out cert.crt -keyout cert.key```

### Tomcat currently operates only on JKS, PKCS11 or PKCS12 format keystores. The following command will create a PKCS12 using the certificate and private key

```openssl pkcs12 -export -in cert.crt -inkey cert.key -out keystore.p12 -name cas -CAfile my_ca_bundle.crt -caname root```

### Create the keystore

```keytool -importkeystore -deststorepass changeit -destkeypass changeit -destkeystore thekeystore -srckeystore keystore.p12 -srcstoretype PKCS12 -srcstorepass changeit -alias cas```

### Import the keystore to our local keystore

```sudo keytool -importkeystore -deststorepass changeit -destkeypass changeit -destkeystore [JAVA_HOME]/jre/lib/security/cacerts  -srckeystore keystore.p12 -srcstoretype PKCS12 -srcstorepass changeit -alias cas```

and restart the app.

### Building the Docker image

This repo provides with shell scripts that makes things a lot easier for us. Before running the CAS Docker image on
any given machine for the first time we need to build the Docker image. Given that the build process require several
steps it is better to do it as an isolated task separated from running a container from the image.

Creating the Docker image is as easy as running the following command:

```./build.sh $CasVersion```

CasVersion is 5.2.0 at the time of this writing. You can check the latest one at [Dockerhub](https://hub.docker.com/r/apereo/cas/tags/).

### Running the Docker image

After building the image, running it is as simple as this:

```./run.sh $CasVersion```

Since some extra configuration is needed to add our service, the image has been changed to run a bash. The next steps has to be done in the container.

### Clean and package the war before adding the services

```./build.sh package```

### Add our service 

```cp etc/cas/services/stockmind.json target/war/work/org.apereo.cas/cas-server-webapp-tomcat/WEB-INF/classes/services/```

### Package the war

```./mvnw package```

### Run the war

```java -jar target/cas.war```

It will take several seconds for the container to start. Once started, we can access the running CAS instance [locally
via port 8443 through HTTPS](https://localhost:8443/cas).
