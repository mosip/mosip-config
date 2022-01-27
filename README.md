# MOSIP Configuration

MOSIP uses Spring CLoud Config Server to read the properties files. So, to use the properties files in this repo, please update the IP addresses, keys and passwords and then point to this repo in spring cloud config in kernel module.

# Local Config Server Installation Guide


## Overview
MOSIP uses Config Server to read the properties files. 


## Download 

1. Download Config server jar [config-server](https://mvnrepository.com/artifact/io.mosip.kernel/kernel-config-server)

2. Clone the Mosip config repo [mosip-config](https://github.com/mosip/mosip-config/tree/develop2-v2)

##Run 

To run config server jar set the following attribute 

```
-Dspring.cloud.config.server.native.search-locations = point to mosip-config repo location

```

### Run Config Server Jar

```
java -jar -Dspring.profiles.active=native  -Dspring.cloud.config.server.native.search-locations=file:C:\mosipcode\mosip-config\sandbox -Dspring.cloud.config.server.accept-empty=true  -Dspring.cloud.config.server.git.force-pull=false -Dspring.cloud.config.server.git.cloneOnStart=false -Dspring.cloud.config.server.git.refreshRate=0 kernel-config-server-1.0.6.jar

```




