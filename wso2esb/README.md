# Mesos Artifacts for WSO2 Enterprise Service Bus

These Mesos Artifacts provide the resources and instructions to deploy WSO2 Enterprise Service Bus on Mesos DC/OS.

## Getting Started

>In the context of this document, `MESOS_HOME`, `DOCKERFILES_HOME` and `PUPPET_HOME` will refer to local copies of [`wso2/mesos-artifacts`](https://github.com/wso2/mesos-artifacts/), [`wso2/dockcerfiles`](https://github.com/wso2/dockerfiles/) and [`wso2/puppet-modules`](https://github.com/wso2/puppet-modules) repositories respectively.

To deploy a WSO2 Enterprise Service Bus on Mesos DC/OS, follow the below steps:

#### 1. Build WSO2 Enterprise Service Bus Docker Image

To manage configurations and artifacts when building Docker images, WSO2 recommends to use [`wso2/puppet-modules`](https://github.com/wso2/puppet-modules) as the provisioning method. A specific data set for Mesos platform is available in WSO2 Puppet Modules. It's possible to use this data set to build Dockerfiles for WSO2 Enterprise Service Bus for Mesos with minimum configuration changes.

Building WSO2 Enterprise Service Bus Docker image using Puppet for Mesos:

  1. Clone `wso2/puppet-modules` and `wso2/dockerfiles` repositories (alternatively you can download the released artifacts using the release page of the GitHub repository).
  2. Add the `mesos-membership-scheme-<version>.jar` file and the dependencies needed to deploy WSO2 Carbon server clusters on Mesos DC/OS to `PUPPET_HOME` as explained in [Mesos DC/OS Membership Scheme for WSO2 Carbon Wiki page](https://docs.wso2.com/display/MA100/Mesos+DC-OS+Membership+Scheme+for+WSO2+Carbon).
  3. Copy the JDK [`jdk-7u80-linux-x64.tar.gz`](http://www.oracle.com/technetwork/java/javase/downloads/jdk7-downloads-1880260.html) to `PUPPET_HOME/modules/wso2base/files` location.
  4. Copy the [`mysql-connector-java-5.1.36-bin.jar`](https://downloads.mysql.com/archives/get/file/mysql-connector-java-5.1.36.zip) to `PUPPET_HOME/modules/wso2esb/files/configs/repository/components/lib` location.
  5. Get the WSO2 Enterprise Service Bus product distribution (4.9.0) and copy it to `PUPPET_HOME/modules/wso2esb/files` location.
  6. Set the environment variable `PUPPET_HOME` pointing to location of the puppet modules in local machine.
  7. Navigate to `wso2esb` directory in the Dockerfiles repository; `DOCKERFILES_HOME/wso2esb`.
  8. Build the Dockerfile with the following command:

    **`./build.sh -v <version> -s mesos -r puppet`**

    Note that `-s mesos` flag denotes the Mesos platform, when it comes to selecting the configuration from Puppet.

    This will build the default profile of WSO2 Enterprise Service Bus for Mesos platform, using configuration specified in Puppet. To deploy the WSO2 Enterprise Service Bus distributed setup, build the `manager` and `worker` profile Docker images using below command:
  
    **`./build.sh -v <version> -s mesos -r puppet -l 'manager|worker'`**
  

#### 2. Load the Docker Images to Mesos slave nodes or import them to Central Docker Registry

Load the required Docker images to Mesos slave nodes(ex: use `docker save` to create a tarball of the required image, `scp` the tarball to each node, and use `docker load` to reload the images from the copied tarballs on the nodes). Alternatively, if a private Docker registry is used, transfer the images there.

You can make use of the `load-images.sh` helper script to transfer images to the Mesos slave nodes. It will search for any Docker images with `mesos` as a part of its name on your local machine, and ask for verification to transfer them to the Mesos slave nodes. `DCOS CLI` has to be functioning on your local machine in order for the script to retrieve the list of Mesos slave nodes. You can optionally provide a search pattern if you want to override the default `mesos` string.

**`./load-images.sh -u centos -p wso2esb -k /home/ssh_key.pem`**

Usage:
```
Usage: ./load-images.sh [OPTIONS]

Transfer Docker images to Mesos Nodes
Options:

  -u	[OPTIONAL] Username to be used to connect to Mesos Nodes. If not provided, default "centos" is used.
  -p	[OPTIONAL] Optional search pattern to search for Docker images. If not provided, default "mesos" is used.
  -k	[OPTIONAL] Optional key file location. If not provided, key file will not be used.
  -h	[OPTIONAL] Show help text.

Ex: ./load-images.sh
Ex: ./load-images.sh -u centos -p wso2is -k /home/ssh_key.pem
```
    
##### 3. Deploy WSO2 Enterprise Service Bus on Mesos DC/OS
  1. Navigate to `wso2esb` directory in mesos-artifacts repository; `MESOS_HOME/wso2esb` location.
  2. Run the deploy.sh script:

    **`./deploy.sh`**
    
    This will deploy following Marathon applications in Mesos DC/OS, using the images available in Mesos slave nodes, and notify once the intended Marathon application `wso2esb-default` starts running on the container.
       * Marathon load balancer
       * WSO2 Governance Registry database
       * WSO2 User Management database
       * WSO2 Enterprise Service Bus Configuration Registry database
       * WSO2 Enterprise Service Bus default profile
       
    To deploy WSO2 Enterprise Service Bus distributed setup on Mesos DC/OS, use `-d` flag with deploy script as below:
     
    **`./deploy.sh -d`**
    
    This will deploy the WSO2 Enterprise Service Bus `wso2esb-manager` and `wso2esb-worker` Marathon applications instead of the default profile, and notify once the Marathon applications starts running on the containers.

#### 4. Login to WSO2 Enterprise Service Bus Management Console
  1. Add a host entry (in Linux, using the `/etc/hosts` file) for Marathon LB Hostname `marathon-lb.marathon.mesos`, resolving to Marathon LB Host IP.
  2. Login to the Carbon Management Console URL using `https://marathon-lb.marathon.mesos:10095/carbon/` in both standalone and distributed deployment.
 
#### 5. Undeploy WSO2 Enterprise Service Bus from Mesos DC/OS
  1. Navigate to `wso2esb` directory in mesos-artifacts repository; `MESOS_HOME/wso2esb` location.
  2. Run the `undeploy.sh` script:

    **`./undeploy.sh`**

    This will undeploy the WSO2 Enterprise Service Bus and its Configuration Registry database(`mysql-esb-db`) Marathon applications.
   
    Additionally if `-f` flag is provided when running `undeploy.sh`, it will also undeploy the shared Governance DB, User DB and Marathon LB applications.
    
    **`./undeploy.sh -f`**

For more detailed instructions on deploying WSO2 Enterprise Service Bus on Mesos DC/OS, please refer the wiki links under the Documentation section below.

# Documentation
* [WSO2 Mesos Artifacts Wiki](https://docs.wso2.com/display/MA100/Home)
