<h1 align>Sonatype Nexus Repository Docker: sonatype/nexus3 🖐</h1>
<h3>🟠Docker composition for Nexus 3 Repository Manager by Sonatype</h3>
<h3>🟠Video about this project https://youtu.be/6WjwrZknYVk 👇</h3>
<img src="README images/1.png" alt="Logo">
<img src="README images/2.png" alt="Logo">
<img src="README images/3.png" alt="Logo">

## Running

To run, binding the exposed port 8081 to the host, use:

```
$ docker run -d -p 8081:8081 --name nexus sonatype/nexus3
```

When stopping, be sure to allow sufficient time for the databases to fully shut down.  

```
docker stop --time=120 <CONTAINER_NAME>
```


To test:

```
$ curl http://localhost:8081/
```

## Building the Sonatype Nexus Repository image

To build a docker image from the [Dockerfile](https://github.com/sonatype/docker-nexus3/blob/main/Dockerfile) you can use this command:

```
$ docker build --rm=true --tag=sonatype/nexus3 .
```

The following optional variables can be used when building the image:

- NEXUS_VERSION: Version of the Sonatype Nexus Repository
- NEXUS_DOWNLOAD_URL: Download URL for Sonatype Nexus Repository, alternative to using `NEXUS_VERSION` to download from Sonatype
- NEXUS_DOWNLOAD_SHA256_HASH: Sha256 checksum for the downloaded Sonatype Nexus Repository archive. Required if `NEXUS_VERSION`
 or `NEXUS_DOWNLOAD_URL` is provided

## Chef Solo for Runtime and Application

Chef Solo is used to build out the runtime and application layers of the Docker image. The Chef cookbook being used is available
on GitHub at [sonatype/chef-nexus-repository-manager](https://github.com/sonatype/chef-nexus-repository-manager).

## Testing the Dockerfile

We are using `rspec` as the test framework. `serverspec` provides a docker backend (see the method `set` in the test code)
 to run the tests inside the docker container, and abstracts away the difference between distributions in the tests
 (e.g. yum, apt,...).

    rspec [--backtrace] spec/Dockerfile_spec.rb

## Red Hat Certified Image

A Red Hat certified container image can be created using [Dockerfile.rh.ubi](https://github.com/sonatype/docker-nexus3/blob/main/Dockerfile.rh.ubi) which is built to be compliant with Red Hat certification.
The image includes additional meta data to comform with Kubernetes and OpenShift standards, a directory with the
licenses applicable to the software and a man file for help on how to use the software. It also uses an ENTRYPOINT
script the ensure the running user has access to the appropriate permissions for OpenShift 'restricted' SCC. 

The Red Hat certified container image is available from the 
[Red Hat Container Catalog](https://access.redhat.com/containers/#/registry.connect.redhat.com/sonatype/nexus-repository-manager)
and qualified accounts can pull it from registry.connect.redhat.com.

## Other Red Hat Images

In addition to the Universal Base Image, we can build images based on:
* Red Hat Enterprise Linux: [Dockerfile.rh.el](https://github.com/sonatype/docker-nexus3/blob/main/Dockerfile.rh.el)
* CentOS: [Dockerfile.rh.centos](https://github.com/sonatype/docker-nexus3/blob/main/Dockerfile.rh.centos)

## Notes

* Our [system requirements](https://help.sonatype.com/display/NXRM3/System+Requirements) should be taken into account when provisioning the Docker container.
* Default user is `admin` and the uniquely generated password can be found in the `admin.password` file inside the volume. See [Persistent Data](#user-content-persistent-data) for information about the volume.

* It can take some time (2-3 minutes) for the service to launch in a
new container.  You can tail the log to determine once Nexus is ready:

```
$ docker logs -f nexus
```

* Installation of Nexus is to `/opt/sonatype/nexus`.  

* A persistent directory, `/nexus-data`, is used for configuration,
logs, and storage. This directory needs to be writable by the Nexus
process, which runs as UID 200.

* There is an environment variable that is being used to pass JVM arguments to the startup script

  * `INSTALL4J_ADD_VM_PARAMS`, passed to the Install4J startup script. Defaults to `-Xms2703m -Xmx2703m -XX:MaxDirectMemorySize=2703m -Djava.util.prefs.userRoot=${NEXUS_DATA}/javaprefs`.

  This can be adjusted at runtime:

  ```
  $ docker run -d -p 8081:8081 --name nexus -e INSTALL4J_ADD_VM_PARAMS="-Xms2703m -Xmx2703m -XX:MaxDirectMemorySize=2703m -Djava.util.prefs.userRoot=/some-other-dir" sonatype/nexus3
  ```

  Of particular note, `-Djava.util.prefs.userRoot=/some-other-dir` can be set to a persistent path, which will maintain
  the installed Sonatype Nexus Repository License if the container is restarted.
  
  Be sure to check the [memory requirements](https://help.sonatype.com/display/NXRM3/System+Requirements#SystemRequirements-MemoryRequirements) when deciding how much heap and direct memory to allocate.

* Another environment variable can be used to control the Nexus Context Path

  * `NEXUS_CONTEXT`, defaults to /

  This can be supplied at runtime:

  ```
  $ docker run -d -p 8081:8081 --name nexus -e NEXUS_CONTEXT=nexus sonatype/nexus3
  ```

### Persistent Data

There are two general approaches to handling persistent storage requirements
with Docker. See [Managing Data in Containers](https://docs.docker.com/engine/tutorials/dockervolumes/)
for additional information.

  1. *Use a docker volume*.  Since docker volumes are persistent, a volume can be created specifically for
  this purpose.  This is the recommended approach.  

  ```
  $ docker volume create --name nexus-data
  $ docker run -d -p 8081:8081 --name nexus -v nexus-data:/nexus-data sonatype/nexus3
  ```

  2. *Mount a host directory as the volume*.  This is not portable, as it
  relies on the directory existing with correct permissions on the host.
  However it can be useful in certain situations where this volume needs
  to be assigned to certain specific underlying storage.  

  ```
  $ mkdir /some/dir/nexus-data && chown -R 200 /some/dir/nexus-data
  $ docker run -d -p 8081:8081 --name nexus -v /some/dir/nexus-data:/nexus-data sonatype/nexus3
  ```

## Getting Help

Looking to contribute to our Docker image but need some help? There's a few ways to get information or our attention:

* Chat with us on [Gitter](https://gitter.im/sonatype/nexus-developers)
* File an issue [on our public JIRA](https://issues.sonatype.org/projects/NEXUS/)
* Check out the [Nexus3](http://stackoverflow.com/questions/tagged/nexus3) tag on Stack Overflow
* Check out the [Sonatype Nexus Repository User List](https://groups.google.com/a/glists.sonatype.com/forum/?hl=en#!forum/nexus-users)

## License Disclaimer

_Sonatype Nexus Repository OSS is distributed with Sencha Ext JS pursuant to a FLOSS Exception agreed upon between Sonatype, Inc. and Sencha Inc. Sencha Ext JS is licensed under GPL v3 and cannot be redistributed as part of a closed source work._
