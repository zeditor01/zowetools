[home](https://github.com/zeditor01/zowetools)

# Deploying ZOWE 

This document is an audit trail of deploying ZOWE V2.17

 ***Contents***

1. Links to ZOWE Documentation
2. Pre-Requisite Software
3. Planning the Deployment
4. Installing the ZOWE product
5. Preparing the zowe.yaml file of parameters
6. Running the 'ZWE INSTALL' Script
7. Running the 'ZWE INIT' Script
8. Starting ZOWE
9. Logging on to ZOWE
10. Using ZOWE
11. ZOWE Automation


## 1. Links to ZOWE Documentation

[zowe project home](https://www.zowe.org/)

[zowe documentation](https://docs.zowe.org/)

[zowe downloads](https://www.zowe.org/download)

 
## 2. Pre-Requisite Software

You must ensure that some prequisite software ( z/OSMF, Java, NodeJS etc...) is installed on z/OS before you install zowe. The main System Requirements are listed here
* AXR (System REXX)
* CEA (Common Event Adapter - of z/OSMF)
* CIM (Common Information Model - of z/OSMF)
* CONSOLE and CONSPROF commands must exist in the authorised command table
* Java (V8 or later)
* NodeJS
* TSO Region Size - minimum 65,536 KB
* Userids - OMVS Segment

Please refer to the current list of pre-requisites [here](https://docs.zowe.org/stable/user-guide/zosmf-install)  before you start.

I am using the May 2024 ADCD build of z/OS (which is based on z/OS V3.1). It included all the pre-requisites, but CFZCIM was not started by default. I changed USER.Z31A.PARMLIB(VTAMALL) to include the ```S CFZCIM``` command.


## 3. Planning the Deployment

The documentation is (imho) difficult to navigate. It comes down to 5 steps.

1. Choose which installation source you will use to install the binaries
2. Edit a yaml file to specify the parameters for your zowe deployment
3. Execute the 'ZWE INSTALL' Script to install Zowe to z/OS
4. Execute the 'ZWE INIT' Script to execute all the customisation steps
5. Start and test your Zowe instance


## 4. Installing the ZOWE product

Zowe has a server-side component (running on z/OS) and an optional client-side component (CLI, SDK, Explorer).

### 4.1 Server Side component

There are 4 options for installing the Zowe server-side component
1. Convenience build (unpack a pax file)
2. SMPE build (FMID + PTFs)
3. Portable Software Instance
4. Containerised build

You can download the server-side component from the Zowe website, or you can order a PSI from ShopZ. I chose the the convenience build. I download V2.17 for this worked example.


With the zowe-convenience build, I downloaded ```zowe-2.17.0.pax``` and transferred it to my smp downloads folder at ```/u/smpe/smpnts/zcb```.

I then allocated a ZFS and mounted it at ```/usr/lpp/zwe/zowe```. Permanent PARMLIB mount specification below.

```
MOUNT FILESYSTEM('ZOWE217.ZFS')       
      TYPE(ZFS)                       
      MODE(RDWR)                      
      NOAUTOMOVE                      
      MOUNTPOINT('/usr/lpp/zwe/zowe') 
```

I then copied the pax file from ```/u/smpe/smpnts/zcb``` zcb into target ZFS ```/usr/lpp/zwe/zowe``` for deployment.

Unpack the pax file with the Command : ```pax -rvf zowe-2.16.0.pax```

Results in
```
IBMUSER:/Z31A/usr/lpp/zwe/zowe: >ls -al
total 1043440
drwxrwxr-x   8 OMVSKERN SYS1        8192 Jul  1 19:33 .
drwxr-xr-x   3 OMVSKERN OMVSGRP     8192 Jul  1 19:21 ..
-rw-r--r--   1 OMVSKERN SYS1        2350 May 23 12:28 DEVELOPERS.md
-rw-r--r--   1 OMVSKERN SYS1        3772 May 23 12:28 README.md
drwxr-xr-x   5 OMVSKERN SYS1        8192 May 23 12:28 bin
drwxr-xr-x  19 OMVSKERN SYS1        8192 May 23 12:30 components
-rw-r--r--   1 OMVSKERN SYS1       25267 May 23 12:28 example-zowe.yaml
drwxr-xr-x   7 OMVSKERN SYS1        8192 May 23 12:30 files
drwxr-xr-x   2 OMVSKERN SYS1        8192 May 23 12:31 fingerprint
drwxr-xr-x   2 OMVSKERN SYS1        8192 May 23 12:28 licenses
-rw-r--r--   1 OMVSKERN SYS1       14213 May 23 12:28 manifest.json
drwxr-xr-x   2 OMVSKERN SYS1        8192 May 23 12:28 schemas
-rw-r-----   1 OMVSKERN SYS1     533836800 Jul  1 19:30 zowe-2.16.0.pax
```

### 4.2 Client Side components

Not covered at this point, because my Db2 tools only need the Server Side component. I will update this guide later when I deploy the CLient Side components.

## 5. Preparing the zowe.yaml file of parameters

The entire Zowe deployment is defined by the zowe.yaml file.

If you've not come across yaml files yet, yaml stands for 'yet another markup language'. The zowe.yaml file is documented [here](https://docs.zowe.org/stable/appendix/zowe-yaml-configuration/). It has 6 sections

1. zowe (Defines global configurations specific to Zowe, including default values)
2. java (Defines Java configurations used by Zowe components)
3. node (Defines node.js configurations used by Zowe components)
4. zOSMF (Tells Zowe your z/OSMF configurations)
5. components (Defines detailed configurations for each Zowe component or extension)
6. haInstances (Defines customized configurations for each High Availability (HA) instance)

I prepared the following zowe.yaml file for my system.


## 6. Running the 'ZWE INSTALL' Script


## 7. Running the 'ZWE INIT' Script


## 8. Starting ZOWE


## 9. Logging on to ZOWE


## 10. Using ZOWE


## 11. ZOWE Automation




