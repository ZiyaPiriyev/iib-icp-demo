# Overview

This repository contains customized Dockerfile and some scripts which demonstrate a way in which you might run [IBM Integration Bus](http://www-03.ibm.com/software/products/en/ibm-integration-bus) in a [Docker](https://www.docker.com/whatisdocker/) container. 

The customizations (@styblope) include options for enabling the embedded global cache in the integration node.

# Building the image

The image can be built using standard [Docker commands](https://docs.docker.com/userguide/dockerimages/) against the supplied Dockerfile.  For example:

~~~
docker build -t iibv10image .
~~~

This will create an image called `iibv10image` occupying approximately 1.15GB of space (including the size of the underlying Ubuntu base image) in your local Docker registry:

~~~
REPOSITORY     TAG       IMAGE ID        CREATED          SIZE
iibv10image    latest    b8403ecfcd0d    2 seconds ago    1.15GB
ubuntu         14.04     132b7427a3b4    3 weeks ago      188MB
~~~

# What the image contains

The built image contains a full installation of [IBM Integration Bus for Developers Edition V10.0](https://ibm.biz/iibdevedn). If you install the stand-alone image, which does not contain an installation of IBM MQ, some functionality may not be available, or may be changed - see this [topic](http://www-01.ibm.com/support/knowledgecenter/SSMKHH_10.0.0/com.ibm.etools.mft.doc/bb28660_.htm) for more information.

# Running a container

After building a Docker image from the supplied files, you can [run a container](https://docs.docker.com/userguide/usingdocker/) which will create and start an Integration Node to which you can [deploy](http://www-01.ibm.com/support/knowledgecenter/SSMKHH_10.0.0/com.ibm.etools.mft.doc/af03890_.htm) integration solutions.


## Running with the default configuration
In order to run a container from this image, it is necessary to accept the terms of the IBM Integration Bus for Developers license.  This is achieved by specifying the environment variable `LICENSE` equal to `accept` when running the image.  You can also view the license terms by setting this variable to `view`. Failure to set the variable will result in the termination of the container with a usage statement.  You can view the license in a different language by also setting the `LANG` environment variable.

In addition to accepting the license, you can optionally specify an Integration Node name using the `NODENAME` environment variable and an Integration Server name using the `SERVERNAME` environment variable.

You can enable the embedded global cache using the `CACHE_POLICY` environment variable. 

The last important point of configuration when running a container from this image, is port mapping.  The Dockerfile exposes ports `4414` and `7800` by default, for Integration Node administration and Integration Server HTTP traffic respectively.  This means you can run with the `-P` flag to auto map these ports to ports on your host.  Alternatively you can use `-p` to expose and map any ports of your choice. The same applies to the image with MQ where the additional port exposed by default is `1414` for the MQ listener.

For example:

~~~
docker run --name myNode -e LICENSE=accept -e NODENAME=MYNODE -e SERVERNAME=MYSERVER -P iibv10image
~~~

If you wish, you can also deploy an IBM Integration Bus BAR file by specifying a [Docker volume](https://docs.docker.com/engine/admin/volumes/volumes/) which makes the BAR file(s) available when the container is started:
~~~
docker run --name myNode -v  /local/path/to/BARs:/tmp/BARs/<yourbars> -e LICENSE=accept -e NODENAME=MYNODE -e SERVERNAME=MYSERVER -P iibv10image 
~~~

This will run a container that creates and starts an Integration Node called `MYNODE` and exposes ports `4414` and `7800` on random ports on the host machine.  At this point you can use:
~~~
docker port <container name>
~~~

to see which ports have been mapped then connect to the Node's web user interface as normal (see [verification](# Verifying your container is running correctly) section below).

## Running with the default configuration and a volume
The above example will not persist any configuration data or messages across container runs.  In order to do this, you need to use a [volume](https://docs.docker.com/engine/admin/volumes/volumes/).  For example, you can create a volume with the following command:

```
docker volume create qm1data
```

You can then run a queue manager using this volume as follows:

```
docker run --name myNode -e LICENSE=accept -e NODENAME=MYNODE -e SERVERNAME=MYSERVER -e MQ_QMGR_NAME=QM1 -v qm1data:/mnt/mqm -P iibv10image
```

The Docker image always uses `/mnt/mqm` for MQ data, which is correctly linked for you under `/var/mqm` at runtime.  This is to handle problems with file permissions on some platforms.

## Accessing logs

This image also configures syslog, so when you run a container, your node will be outputting messages to /var/log/syslog inside the container.  You can access this by attaching a bash session as described above or by using docker exec.  For example:

~~~
docker exec <container id> tail -f /var/log/syslog
~~~

# Verifying your container is running correctly

Whether you are using the image as provided or if you have customised it, here are a few basic steps that will give you confidence your image has been created properly:

1. Run a container, making sure to expose port 4414 to the host - the container should start without error
2. Run mqsilist to show the status of your node as described above - your node should be listed as running
3. Access syslog as descried above - there should be no errors
4. Connect a browser to your host on the port you exposed in step 1 - the Integration Bus web user interface should be displayed.

At this point, your container is running and you can [deploy](http://www-01.ibm.com/support/knowledgecenter/SSMKHH_10.0.0/com.ibm.etools.mft.doc/af03890_.htm) integration solutions to it using any of the supported methods.


# List of all Environment variables supported by this image

* **LICENSE** - Set this to `accept` to agree to the MQ Advanced for Developers license. If you wish to see the license you can set this to `view`.
* **LANG** - Set this to the language you would like the license to be printed in.
* **NODENAME** - Set this to the name you want your Integration Node to be created with.
* **SERVERNAME** - Set this to the name you want your Integration Server to be created with.
* **CACHE_POLICY** - Set this to configure the embedded global cache. The parameter specifies the policy to use for the cache manager. You can set this parameter to `default`, `disabled`, `none`, or the fully qualified name of an XML policy file. 
* **CACHE_PORT_RANGE** - Specify the range of ports that the cache manager can use. Set this parameter to `generate` or to a specific range of ports. If you specify a range of ports, the value of this parameter must be in the format `xxxx-yyyy`, and the range must contain at least 20 ports


# License

The Dockerfile and associated scripts are licensed under the [Eclipse Public License 1.0](./LICENSE). Licenses for the products installed within the images are as follows:

 - IBM Integration Bus for Developers is licensed under the IBM International License Agreement for Non-Warranted Programs. This license may be viewed from the image using the `LICENSE=view` environment variable as described above.
 - IBM MQ Advanced for Developers is licensed under the IBM International License Agreement for Non-Warranted Programs. This license may be viewed from the image using the `LICENSE=view` environment variable as described above.
 - License information for Ubuntu packages may be found in `/usr/share/doc/${package}/copyright`

Note that the IBM Integration Bus for Developers license does not permit further distribution. The IBM MQ Advanced for Developers license does not permit further distribution and the terms restrict usage to a developer machine.