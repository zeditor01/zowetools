# Deploying Unified Management Server (UMS)

This document is an audit trail of deploying UMS V1.2

UMS is a pre-requisite for the zowe-based Db2 experiences.

UMS is a ZOWE plugin that provides a foundation for multiple DB2 and IMS tools. You must install UMS before you can install Db2 Administration Foundation and other experiences.

UMS provides common services that are used by other tools, such as
* Credential management by UMS JWT tokens
* Credential management by Zowe JWT tokens (to support MFA)
* JDBC connections by a DBA userid to IMS and DB2 subsystems
* Catalog Navigation Services to IMS and DB2 subsystems
* etc...

The knowledge center illustrates the relationship between UMS and ZOWE as follows

![ums_arch](/images/ums_arch.jpg)

 ***Contents***

1. Links to UMS Documentation
2. Pre-Requisite Software
3. Planning the Deployment
4. Installing the UMS product
5. Post-SMPE Installation of UMS
   5.1 collecting Required Parameters
   5.2 Stopping ZOWE and ZSS
   5.3 Copy SIZPSAMP
   5.4 IZPALOPL
   5.5 IZPCPYML
   5.6 Edit ZWEYAML
   5.7 IZPGENER
   5.8 IZPA3
   5.9 Execute Selected  ESM JCLs
   5.10 Encrypt DBA credentials
   5.11 IZPIPLUG
   5.12 IZPEXPIN
   5.13 Edit ZOWE STC JCL
   5.14 Start ZSS
   5.15 Start ZOWE
6. Validate ZOWE with UMS






## 1. Links to UMS Documentation

The UMS Knowledgecenter is [here](https://www.ibm.com/docs/en/umsfz/1.2.0)

## 2. Pre-Requisite Software

The hardware and software pre-requisites for UMS and the Db2 experiences are documented [here](https://www.ibm.com/docs/en/umsfz/1.2.0?topic=installation-prerequisite-hardware-software)

In addition to z/OS V2.4 or later, ICSF and RACF, minimum versions of ZOWE, and a number of PTF levels are documented at the link above. You should check the pre-requisites before proceeding.


## 3. Planning the Deployment

UMS installation comprises

1. ShopZ PSI installation
2. Post-Installation Steps to establish some deployment libraries
3. Edit a YAML file to define the parameters that control UMS, and to integrate with your Zowe instance
4. Generate a series of customisation jobs to establish the RACF artefacts to manage UMS
5. Execute those jobs
6. Start Zowe, with references to UMS

The challenges in installing UMS are primarily associated with RACF certificates and permissions and userids and integration points with ZOWE. It is worth taking a moment to understand the authentication flows for UMS, as shown by the diagram below from the [knowledge center](https://www.ibm.com/docs/en/umsfz/1.2.0?topic=zos-credential-management-by-ums-jwt-tokens).

![ums_jwt](/images/ums_jwt.jpg)

UMS and ZOWE do provide support for MFA tokens, documented [here](https://www.ibm.com/docs/en/umsfz/1.2.0?topic=zos-credential-management-by-zowe-jwt-tokens), but that is outside the scope of this worked example.

## 4. Installing the ZOWE product

The PSI installation is outside the scope of this document. I performed a standard SMPE PSI installation to HLQ "IZP", resulting in the following z/OS Datasets
```
Data Set Names / Objects                       Volume
---------------------------------------------- ------
'IZP.AIZPBASE'                                 A3USR8
'IZP.AIZPBIN'                                  A3USR3
'IZP.AIZPCUSA'                                 A3USR4
'IZP.AIZPCUSC'                                 A3USR3
'IZP.AIZPCUSR'                                 A3USR4
'IZP.AIZPCUST'                                 A3USR8
'IZP.AIZPLOAD'                                 A3USR3
'IZP.AIZPPARM'                                 A3USR5
'IZP.AIZPPAX'                                  A3USR2
'IZP.AIZPREXX'                                 A3USR2
'IZP.AIZPSAMP'                                 A3USR4
'IZP.CPAC.DB2.PARMLIB'                         A3USR4
'IZP.CPAC.PDFPD'                               A3USR7
'IZP.CPAC.PROD.PDF'                            A3USR2
'IZP.CPAC.PROPVAR'                             A3USR2
'IZP.CPAC.SCPPCENU'                            A3USR7
'IZP.CPAC.SCPPLOAD'                            A3USR6
'IZP.CPAC.WORKFLOW'                            A3USR6
'IZP.CPACDB2.DOCLIB'                           A3USR2
'IZP.CPACDB2.SAMPLIB'                          A3USR6
'IZP.DBA.ENCRYPT'                              A3USR2
'IZP.ENVIRON'                                  A3USR5
'IZP.JCLLIB'                                   A3USR3
'IZP.OMVS.SIZPROOT'                                  
'IZP.OMVS.SIZPROOT.DATA'                       A3USR5
'IZP.PARMLIB'                                  A3USR2
'IZP.SAFXDBRM'                                 A3USR2
'IZP.SAFXLLIB'                                 A3USR6
'IZP.SAFXSAMP'                                 A3USR7
'IZP.SAMPLIB'                                  A3USR3
'IZP.SIZPBASE'                                 A3USR2
'IZP.SIZPCUSA'                                 A3USR7
'IZP.SIZPCUSC'                                 A3USR3
'IZP.SIZPCUSR'                                 A3USR4
'IZP.SIZPCUST'                                 A3USR6
'IZP.SIZPLOAD'                                 A3USR4
'IZP.SIZPPARM'                                 A3USR3
'IZP.SIZPREXX'                                 A3USR8
'IZP.SIZPSAMP'                                 A3USR3
'IZP.SMPE.DB2.DLIB.CSI'                              
'IZP.SMPE.DB2.DLIB.CSI.DATA'                   A3USR6
'IZP.SMPE.DB2.DLIB.CSI.INDEX'                  A3USR6
'IZP.SMPE.DB2.GLOBAL.CSI'                            
'IZP.SMPE.DB2.GLOBAL.CSI.DATA'                 A3USR6
'IZP.SMPE.DB2.GLOBAL.CSI.INDEX'                A3USR6
'IZP.SMPE.DB2.SMPGLOG'                         A3USR4
'IZP.SMPE.DB2.SMPGLOGA'                        A3USR8
'IZP.SMPE.DB2.SMPPTS'                          A3USR4
'IZP.SMPE.DB2.TARGET.CSI'                            
'IZP.SMPE.DB2.TARGET.CSI.DATA'                 A3USR2
'IZP.SMPE.DB2.TARGET.CSI.INDEX'                A3USR2
'IZP.SMPE.DB2D300.SMPDLOG'                     A3USR3
'IZP.SMPE.DB2D300.SMPDLOGA'                    A3USR5
'IZP.SMPE.DB2T300.SMPMTS'                      A3USR7
'IZP.SMPE.DB2T300.SMPSCDS'                     A3USR2
'IZP.SMPE.DB2T300.SMPSTS'                      A3USR6
'IZP.SMPE.DB2T300.SMPTLOG'                     A3USR3
'IZP.SMPE.DB2T300.SMPTLOGA'                    A3USR4
'IZP.TEAMLIST'                                 A3USR8
'IZP.USERLIST'                                 A3USR7
   60 'IZP.USERLIST'                                 A3USR7 
```

There is a ZFS filesystem that I mounted permenantly in PARMLIB as follows
```                               
MOUNT FILESYSTEM('UMS.OMVS.SIZPROOT')            
      TYPE(ZFS)                                  
      MODE(RDWR)                                 
      NOAUTOMOVE                                 
      MOUNTPOINT('/usr/lpp/IBM/izp/v1r2m0/bin')  
```

And the contents of the ZFS are as follows
```
IBMUSER:/Z31A/usr/lpp/IBM/izp/v1r2m0/bin: >ls -al
total 274016
drwxr-xr-x  17 OMVSKERN SYS1        8192 Jul  4 22:23 .
drwxr-xr-x   3 OMVSKERN OMVSGRP     8192 Jul  4 22:33 ..
drwxr-xr-x   2 OMVSKERN OMVSGRP     8192 Jun 13 05:25 IBM
-rwxrwxr-x   2 OMVSKERN OMVSGRP     1425 Jun 13 05:25 IZPCP
-rwxrwxr-x   2 OMVSKERN OMVSGRP     7035 Jun 13 05:25 IZPUNPAX
-rwxrwxr-x   2 OMVSKERN OMVSGRP     6088 Jun 13 05:25 IZPUNPX2
drwxrwxr-x   3 OMVSKERN OMVSGRP     8192 Nov 28  2022 cidb
-rwxrwxr-x   2 OMVSKERN OMVSGRP   483840 Jun 13 05:25 cidb.pax
drwxr-xr-x   3 OMVSKERN OMVSGRP     8192 Dec 15  2023 db2-auth
-rwxrwxr-x   2 OMVSKERN OMVSGRP    32320 Jun 13 05:25 db2-auth.pax
drwxr-xr-x   5 OMVSKERN OMVSGRP     8192 Dec 18  2023 ivp
-rwxrwxr-x   2 OMVSKERN OMVSGRP   516160 Jun 13 05:25 ivp.pax
-rwxrwxr-x   2 OMVSKERN OMVSGRP  89349120 Jun 13 05:25 izpserve.pax
-rwxrwxr-x   2 OMVSKERN OMVSGRP     1153 Jun 13 05:25 manifest.yaml
drwxr-xr-x   2 OMVSKERN OMVSGRP     8192 Dec  1  2022 openssl
-rwxrwxr-x   2 OMVSKERN OMVSGRP  26417680 Jun 13 05:25 openssl.pax
drwxr-xr-x   3 OMVSKERN OMVSGRP     8192 Nov 28  2022 platform
-rwxrwxr-x   2 OMVSKERN OMVSGRP    32320 Jun 13 05:25 platform-activation.pax
drwxr-xr-x   4 OMVSKERN OMVSGRP     8192 Dec 18  2023 ums
drwxr-xr-x   4 OMVSKERN OMVSGRP     8192 Jun 21  2023 ums-security
-rwxrwxr-x   2 OMVSKERN OMVSGRP   483840 Jun 13 05:25 ums-security.pax
drwxr-xr-x   7 OMVSKERN OMVSGRP     8192 Dec 15  2023 unified-ui
-rwxrwxr-x   2 OMVSKERN OMVSGRP  17224720 Jun 13 05:25 unified-ui.pax
drwxr-xr-x   4 OMVSKERN OMVSGRP     8192 Sep 21  2023 zos-newton-daj
-rwxrwxr-x   2 OMVSKERN OMVSGRP   838720 Jun 13 05:25 zos-newton-daj.pax
drwxrwxr-x   4 OMVSKERN OMVSGRP     8192 Nov 28  2022 zos-newton-db2ifi
-rwxrwxr-x   2 OMVSKERN OMVSGRP   612880 Jun 13 05:25 zos-newton-db2ifi.pax
drwxr-xr-x   4 OMVSKERN OMVSGRP     8192 Aug  4  2023 zos-newton-discovery
-rwxrwxr-x   2 OMVSKERN OMVSGRP  1548320 Jun 13 05:25 zos-newton-discovery.pax
drwxrwxr-x   4 OMVSKERN OMVSGRP     8192 Nov 28  2022 zos-newton-registry
-rwxrwxr-x   2 OMVSKERN OMVSGRP   548400 Jun 13 05:25 zos-newton-registry.pax
drwxrwxr-x   4 OMVSKERN OMVSGRP     8192 Nov 28  2022 zos-newton-security
-rwxrwxr-x   2 OMVSKERN OMVSGRP   612880 Jun 13 05:25 zos-newton-security.pax
drwxr-xr-x   4 OMVSKERN OMVSGRP     8192 Sep 21  2023 zss-data-provider
-rwxrwxr-x   2 OMVSKERN OMVSGRP  1193520 Jun 13 05:25 zss-data-provider.pax
```

Check the UMS documentation for SMPE tasks [here](https://www.ibm.com/docs/en/umsfz/1.2.0?topic=begin-performing-smpe-installation-tasks)

UMS Zowe plug-ins require Program Control authorization. In order to tag the files with this bit, the SMP/E install user requires BPX.FILEATTR.PROGCTL permission on the system.

```
extattr +p */zssServer/lib/*
```

So, I opened an ssh terminal, and navigated to the ```components.izp.runtimeDirectory``` directory, and executed the commad

```
IBMUSER:/Z31A/usr/lpp/IBM/izp/v1r2m0/bin: >pwd
/Z31A/usr/lpp/IBM/izp/v1r2m0/bin
IBMUSER:/Z31A/usr/lpp/IBM/izp/v1r2m0/bin: >extattr +p */zssServer/lib/*
IBMUSER:/Z31A/usr/lpp/IBM/izp/v1r2m0/bin: >
```


## 5. Post-SMPE Installation of UMS

The [UMS Knowledgecenter}(https://www.ibm.com/docs/en/umsfz/1.2.0?topic=installation-post-smpe-ums) does a pretty good job of explaining the customisation steps for UMS, including the overview of the workflow

![ums_workflow](/images/ums_workflow.JPG)

### 5.1 collecting Required Parameters

Refer to the documentation for [collect_params](https://www.ibm.com/docs/en/umsfz/1.2.0?topic=ums-step-1-collecting-required-parameters#izp_ig_con_installing__USS_Folder)

The UMS installation script uses the PARMLIB(ZWEYAML) member that is created by IZPCPYML. For details of the IZPCPYML JCL and the PARMLIB member ZWEYAML, see Step 2: Installing Unified Management Server. For the notation of ZWEYAML parameters that are referred to in the table, see YAML format.

* Item 1: Prepare a DBA user ID and ensure its privileges
* Item 2: Identify the UMS z/OS UNIX System Services directories
* Item 3: Confirm the location of Zowe configuration YAML file
* Item 4: Validate Integrated Cryptographic Service Facility (ICSF)
* Item 5: Determine locations for UMS data sets

JCL to prepare a DBA userid
```
//IBMUSERR JOB (ACCOUNT),'NEALE ARMSTRONG',NOTIFY=&SYSUID,
// MSGCLASS=H,CLASS=A,MSGLEVEL=(1,1),REGION=0M            
//S1       EXEC PGM=IKJEFT01                                                  
//SYSTSPRT DD SYSOUT=*                                    
//SYSPRINT DD SYSOUT=*                                    
//SYSTSIN  DD *                                           
ADDUSER IZPDBA1 NAME('IZPDBA1')     +                     
  OMVS(AUTOUID HOME('/u/izpdba1')   +                     
  PROGRAM('/bin/sh')) DFLTGRP(SYS1) NOPASSWORD            
/*                                                        
```



### DBA ID
When you plan to use a protected user as the DBA user ID and client authentication with user mapping for the DBA user, you need to prepare a key ring and a client certificate for the DBA user ID. omvs segment + sysadm privilege. authenticaton either bu uid & encrypted pwd token, or by certificate as covered [here](https://www.ibm.com/docs/en/umsfz/1.2.0?topic=begin-setting-up-dba-user-unified-management-server)

### USS Directories

You must identify the USS directory where UMS is installed. 
Note that this is the SMP/E directory (/usr/lpp/IBM/izp/v1r2m0/bin) which is already installed . 
This directory will be referred to as components.izp.runtimeDirectory in the rest of this document. 

The UMS started task ID is same as the Zowe started task ID. 
The ID must have read/write access to the directory that will be specified by the parameter components.izp.workspaceDirectory in the member ZWEYAML of the UMS PARMLIB. 
This directory and its sub directories will be used as the UMS workspace where UMS files to be installed, added, or updated will be placed. 

### zowe YAML file
Confirm in which USS directory the Zowe configuration YAML file (zowe.yaml) for the Zowe instance to be used for the UMS server installation is located. (/usr/lpp/zwe/zowe/zowe.yaml)
The zowe.yaml file location will be specified in the IZPGENER JCL to be used in the UMS server installation. 

### ICSF

RACF Keyrings and/or ICSF

### Datasets
* HLQ for UMS data sets. (UMS.IZP)
* PARMLIB (UMS needs a partitioned dataset to store its parameter files. There will be a main UMS PARMLIB member called ZWEYAML and additional PARMLIB members for each installed experience.)
* JCLLIB (UMS creates a number of JCL jobs from a template that is installed by SMP/E based on the values in your PARMLIB.)


   
### 5.2 Stopping ZOWE and ZSS

Stop all ZWE* address spaces from SDSF

### 5.3 Copy SIZPSAMP



### 5.4 IZPALOPL


### 5.5 IZPCPYML


### 5.6 Edit ZWEYAML


### 5.7 IZPGENER


### 5.8 IZPA3


### 5.9 Execute Selected  ESM JCLs


### 5.10 Encrypt DBA credentials


### 5.11 IZPIPLUG


### 5.12 IZPEXPIN


### 5.13 Edit ZOWE STC JCL


### 5.14 Start ZSS


### 5.15 Start ZOWE


## 6. Starting ZOWE with UMS
