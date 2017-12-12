# IBM Content Search Services Overview
IBM® Content Search Services container is a Docker image that enables you to quickly deploy IBM Content Search Services. The Content Search Services container image is based on IBM Content Search Services v5.5.0.

# Limitations
Like the on-premise version, IBM Content Search Services servers still needs to be registered in the Administration Console for Content Platform Engine.

# Requirements and prerequisites
Before you deploy and run the IBM Content Search Services container image, confirm the following prerequisites:

- A Docker runtime environment (a Linux host or virtual machine with Docker installed)
- IBM FileNet P8 Content Platform Engine (CPE) container, deployed and configured

# Preparing for container installation

## 1. Secure the port.

- The communication between Content Platform Engine and Content Search Services is in SSL, and the default secure port is 8199. Non-SSL communication is disabled. Users can specify a different host port when running Content Search Services container, for example:

```
# Publish CSS container's port 8199 to the host 8199
# when running docker container named cssx1
docker run -d --name cssx1 -p 8199:8199 ****
# Publish CSS container's port 8199 to the host 8200
# when running docker container named cssx2
docker run -d --name cssx2 -p 8200:8199 ****
```

## 2. Create shared volumes and mount points.

- Create folders on host shared storage to hold the deployment-specific configuration files as well as data that will live outside the container, for data persistence. 

Container directory sample | Host directory example | Description
------------ | ------------- | ------------------------
/opt/IBM/ContentSearchServices/CSS_Server/data | /home/css/CSS_Server_data | Data persistence for CSS config data and SSL sample keystore |
/opt/IBM/ContentSearchServices/CSS_Server/log | /home/css/CSS_Server_log | Data persistence for CSS log files | 
/opt/IBM/ContentSearchServices/CSS_Server/temp | /home/css/CSS_Server_temp | Data persistence for CSS temp files
/CSSIndex1_OS1 | /CSSIndex1_OS1 | Data persistence for content index data

For example:
```
# Mount CSS data and log directory when starting container:
docker run **** 
-v /home/css/CSS_Server_data:/opt/IBM/ContentSearchServices/CSS_Server/data
-v /home/css/CSS_Server_log:/opt/IBM/ContentSearchServices/CSS_Server/log
-v /home/css/CSS_Server_temp:/opt/IBM/ContentSearchServices/CSS_Server/temp
-v /CSSIndex1_OS1:/CSSIndex1_OS1
```


Container type | Host directory example | Run container command sample
------------ | ------------- | ------------------------
CPE (Content Platform Engine) | /CSSIndex1_OS1  | docker run -v /CSSIndex1_OS1:/CSSIndex1_OS1 **** CPE_IMAGE
CSS (Content Search Services) | /CSSIndex1_OS1  | docker run -v /CSSIndex1_OS1:/CSSIndex1_OS1 **** CSS_IMAGE

## 3. Set owner permission on the host directories.
- Because services run with a non-root user (`uid=501` and `gid=500`), you must set owner permission on these host mount directories. Note that the UID of 501 is a fixed value whether or not it exists on the host. 
For example:

```
# uid must be 501 and gid must be 500
# Change owner on host CSS data directory:
chown -R 501:500 /home/css/CSS_Server_data
# Change owner on host CSS log directory:
chown -R 501:500 /home/css/CSS_Server_log
# Change owner on host CSS temp directory:
chown -R 501:500 /home/css/CSS_Server_temp
# Change owner on host content index data directory:
chown -R 501:500 /CSSIndex1_OS1
```

## 4. Configure the SSL keystore.
- A sample SSL keystore file [cssSelfsignedServerStore](https://github.com/ibm-ecm/container-css/blob/master/sample/cssSelfsignedServerStore) is provided for the Content Search Services container. The certification inside the sample SSL keystore of the Content Search Services container are same as the certificaton inside the default sample keystores inside the Content Platform Engine container, to ensure SSL communication work well between these containers.

Sample SSL keystore in the Content Search Services container mount path

            /opt/IBM/ContentSearchServices/CSS_Server/data/cssSelfsignedServerStore

Sample SSL keystores inside the Content Platform Engine container

            /opt/ibm/wlp/usr/servers/defaultServer/resources/security/keystore.jks
            /opt/ibm/wlp/usr/servers/defaultServer/resources/security/truststore.jks

- Download and copy the sample SSL keystore file [cssSelfsignedServerStore](https://github.com/ibm-ecm/container-css/blob/master/sample/cssSelfsignedServerStore) to the mount path of the CSS_Server data. When you run the Content Search Services container, it communicates with Content Platform Engine in SSL by using the sample keystore file. For example:

```
# Put sample SSL keystore to host CSS data path
/home/css/CSS_Server_data/cssSelfsignedServerStore
```
- You can also choose to use customized certifications in the keystore file. See [this readme](https://github.com/ibm-ecm/container-css/blob/master/Customize_SSL_cert_CPE_CSS.md) for details. 


# Quickstart
## 1. Pull the Content Search Services Docker image.
Use the following command with your own credentials.

```
docker login -u ***** -p ****** 
docker pull ecmcontainers/ecm_earlyadopters_css:earlyadopters-gm5.5
```
## 2. Set the environment variables (optional).
These optional variables can be set on your Docker container. You can pass these to the container when you run the image by using additional -e FLAG=value arguments.

Name | Description | Required | Default Value
------------ | ------------- | ------------- | -------------
JVM_HEAP_XMX | Maximum Java heap size | No | 3072

The JVM_HEAP_XMX can also be adjusted during CSS container run time. Run the following command on the Docker host where the Content Search Services container is running to change the value of JVM_HEAP_XMX dynamically.

```
docker exec -d [CSS container name] /usr/bin/css-dynamic-change.sh [JVM_HEAP_XMX value]
e.g.
docker exec -d cssx1 /usr/bin/css-dynamic-change.sh 4096
```

## 3. Run the container in the Docker environment.

A Linux host or virtual machine with Docker engine installed is required to run this image. You can refer to the Usage section for details. The following command runs the Content Search Services container using sample values:

```
docker run -d --name CSSX1 -p 8199:8199 --hostname=cssx1 -v /home/css/CSS_Server_data:/opt/IBM/ContentSearchServices/CSS_Server/data -v /home/css/CSS_Server_log:/opt/IBM/ContentSearchServices/CSS_Server/log -v /home/css/CSS_Server_temp:/opt/IBM/ContentSearchServices/CSS_Server/temp -v /CSSIndex1_OS1:/CSSIndex1_OS1 ecmcontainers/ecm_earlyadopters_css:earlyadopters-gm5.5
```

# Usage
## Run the Content Search Service container.
> Usage: docker run -d --name [CONTAINER NAME] -p {Listening port on docker host}:{Content Search Service port} -v {css data host volume}:{css data container directory} -v {CSS log host volume}:{CSS log container directory} -v {CSS temp host volume}:{CSS temp container directory} -v {Content index data host volume}:{Content index data container directory} IMAGE


### Options
- <b>-p {Listening port on docker host}:{Content Search Service port}</b>
<br>Specify the Content Search Service listening port on docker host (Default Content Search Service security port is 8199)
- <b>-v {CSS data host volume}:{CSS data container directory}</b>
<br>Mount CSS data volume.
- <b>-v {CSS log host volume}:{CSS log container directory}</b>
<br>Mount CSS log volume.
- <b>-v {CSS temp host volume}:{CSS temp container directory}</b>
<br>Mount CSS temp volume.
- <b>-v {Content index data host volume}:{Content index data container directory}</b>
<br>Mount Content index data volume.

> For example:
>> docker run -d --name CSSX1 -p 8199:8199 --hostname=cssx1 -v /home/css/CSS_Server_data:/opt/IBM/ContentSearchServices/CSS_Server/data -v /home/css/CSS_Server_log:/opt/IBM/ContentSearchServices/CSS_Server/log -v /home/css/CSS_Server_temp:/opt/IBM/ContentSearchServices/CSS_Server/temp -v /CSSIndex1_OS1:/CSSIndex1_OS1 ecmcontainers/ecm_earlyadopters_css:earlyadopters-gm5.5

## Run with monitoring enabled.
You can also choose to start the Content Search Service container with monitoring enabled by using a different Docker run command line. With monitoring enabled, you can monitor container resource and logs with monitoring server

For monitoring environment variables, please check [ECM Monitoring Github](https://github.com/ibm-ecm/container-monitoring#environment-variables).

Run Content Search Service container with Bluemix monitoring
> For example:
>> docker run -d --name CSSX1 -p 8199:8199 -e MON_METRICS_WRITER_OPTION=2 -e MON_METRICS_SERVICE_ENDPOINT=metrics.ng.bluemix.net:9095 -e MON_BMX_GROUP=com.ibm.ecm.monitor. -e MON_BMX_METRICS_SCOPE_ID={space or organization guid} -e MON_BMX_API_KEY={IAM API key} -e MON_LOG_SHIPPER_OPTION=2 -e MON_BMX_SPACE_ID={tenant id} -e MON_LOG_SERVICE_ENDPOINT=logs.opvis.bluemix.net:9091 -e MON_BMX_LOGS_LOGGING_TOKEN={log logging token} --hostname=cssx1 -v /home/css/CSS_Server_data:/opt/IBM/ContentSearchServices/CSS_Server/data -v /home/css/CSS_Server_log:/opt/IBM/ContentSearchServices/CSS_Server/log -v /home/css/CSS_Server_temp:/opt/IBM/ContentSearchServices/CSS_Server/temp -v /CSSIndex1_OS1:/CSSIndex1_OS1 ecmcontainers/ecm_earlyadopters_css:earlyadopters-gm5.5  

## Configuring on Content Platform Engine for Content Search Services
- Get Content Search Service authentication token
The authentication token is used to communicate with the CPE server. Record this authentication token string value; it is used in text search configuration in Content Platform Engine.
    * Connect to the running Content Search Service Docker container:
    
            docker exec -it [CSS Container Name] /bin/bash
    * Execute Content Search Service configTool to get token information:

            cd /opt/IBM/ContentSearchServices/CSS_Server/bin
            ./configTool.sh printToken -configPath ../config
    * Check the output and store the authentication token:

            The authentication token is printed below. This token is used to communicate with the server. Store the token if applicable.
            RNUNEWc=
            The encryption key is printed below. This key is used to encrypt the password during text index backup and restore operations. Store the key if applicable.
            RNUNEWd4rc35R80IsYNLTg==

- Create the Content Search Service server in the Administration Console for Content Platform Engine:
    * Log in to the Administration Console and navigate to Domain > Global Configuration > Administration > Text Search Servers > New.
    * Define the Text Search Server property values as follows:
    
            Mode: Dual: Index and Search
            Status: Enabled
            Host name: {docker host name where CSS container running}
            Port: {listening security port on docker host}
            Authentication token: {The CSS authentication token, "RNUNEWc=" for the example}
            Set the Is SSL Enabled field value to True.
            Set the Validate Server Certificate and the Validate Certificate Host field values to False.
 
 - Configure the CPE object store
    * Edit the object store to be configured > Select Text Search tab
    
            1. Checked on option of Enable IBM Content Search Services
            2. Select one or more Indexing Languages, and set default language (default is en)
    
    * Create Index Area: Object Store > Administrative > Index Areas. The text search index data will be stored in the Root directory on the Text Search Server.
    
            Create one index area and set Root directory
            e.g. /CSSIndex1_OS1
            
    * Enable the class for full text search: Object Store > Data Design > Classes.
    
            1. Edit the class to be enabled for full text indexing
            2. Checked “CBR enabled” option

## Run the IBM Content Search Services container on Kubernetes.  

1. Prepare persistence volumes:  

Refer to [kubernetes document](https://kubernetes.io/docs/concepts/storage/persistent-volumes/) for information on persistence volume preparation.

2. Create a persistence volume claim ([sample YAML file for create PVC](https://github.com/ibm-ecm/container-icn/tree/master/examples/ecmcfgstore.yml)):

```
kubectl apply ecmcfgstore.yml
```

Use the following command to check persistence volume information:

```
kubectl describe pvc <PVC_NAME>
```

You can use the command to bind the persistence volume name to this persistence volume claim.

3. Provision data in the storage volume.
- Mount the configuration storage on the Kubernetes client:

```
mkdir /cfgstore
mount -t nfs4 -o hard,intr <PV_HOST>:/<PV_FOLDER> /cfgstore
```

Where PV_HOST is the server name of the persistence volume, and PV_FOLDER is the path for the persistence volume. You can check these values by running the following command:

```
kubectl describe pv <PV_NAME>  
```

- Create the following folders under /cfgstore

```
    /cfgstore/css/css_server_data
    
    /cfgstore/css/css_server_logs

    /cfgstore/css/css_server_temp

    /indexstore/CSSIndexArea1_OS1
    
```

- Copy the sample SSL keystore file [cssSelfsignedServerStore](https://github.com/ibm-ecm/container-css/blob/master/sample/cssSelfsignedServerStore) or customized keystore file to /cfgstore/css/css_server_data

4. Deploy IBM Content Search Services ([sample YAML file](https://github.com/ibm-ecm/container-css/blob/master/sample/css-deploy.yaml)).

```
kubectl create -f css-deploy.yaml
```

# Support
Support can be obtained at [IBM® DeveloperWorks Answers](https://developer.ibm.com/answers/)
<br>
Use the ECM-CONTAINERS tag and assistance will be provided.<br>
*Note: Limited support available during Early Adopter Program*
