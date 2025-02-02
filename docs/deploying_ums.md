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
MOUNT FILESYSTEM('IZP.OMVS.SIZPROOT')            
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

We need to get all the parameters and artefacts ready, so that we can edit the YAML ```IZP.PARMLIB(ZWEYAML)``` for for UMS, and generate and execute the customisation jobs. The items to prepare are as follows

* Item 1: Prepare a DBA user ID and ensure its privileges
* Item 2: Identify the UMS z/OS UNIX System Services directories
* Item 3: Confirm the location of Zowe configuration YAML file
* Item 4: Validate Integrated Cryptographic Service Facility (ICSF)
* Item 5: Determine locations for UMS data sets

### 5.1.1 DBA ID
When you plan to use a protected user (i.e. a user with heightened powers and without tso logon) as the DBA user ID and client authentication with user mapping for the DBA user, you need to prepare a key ring and a client certificate for the DBA user ID. omvs segment + sysadm privilege. authenticaton either bu uid & encrypted pwd token, or by certificate as covered [here](https://www.ibm.com/docs/en/umsfz/1.2.0?topic=begin-setting-up-dba-user-unified-management-server)

JCL I used to prepare a DBA userid ... because the JCL generated by IZPGENER doesn't work .... IZP.JCLLIB(IZPD4R) 
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

I'm pretty sure that ADCD includes ICSF

### Datasets
* HLQ for UMS data sets. (UMS.IZP)
* PARMLIB (UMS needs a partitioned dataset to store its parameter files. There will be a main UMS PARMLIB member called ZWEYAML and additional PARMLIB members for each installed experience.)
* JCLLIB (UMS creates a number of JCL jobs from a template that is installed by SMP/E based on the values in your PARMLIB.)

### Other Notes

Identify the UMS z/OS UNIX System Services folder

/usr/lpp/IBM/izp/v1r2m0/bin

/usr/lpp/zwe/zwe217/zowe.yaml 

/usr/lpp/IBM/afx/v1r2m0/bin	AFX.OMVS.SAFXROOT

Validate Integrated Cryptographic Service Facility (ICSF)
Determine locations for UMS data sets
Identify users who will need access to UMS


   
### 5.2 Stopping ZOWE and ZSS

Stop all ZWE* address spaces from SDSF

### 5.3 Copy SIZPSAMP

Allocate IZP.SAMPLIB as a PDS.

Copy IZP.SIZPSAMP(*) to IZP.SAMPLIB 

IZP.SAMPLIB Contains members below
```
IZPALOPL
IZPCPYML
IZPCPYM2
IZPGENER
IZPMIGRA
IZPSYNCY
```

### 5.4 IZPALOPL

Execute IZP.SAMPLIB(IZPLOPL) - allocate new parmlib "IZP.PARMLIB"
```
//IZPALOPL EXEC PGM=IEFBR14                   
//ALLOC    DD   DSN=IZP.PARMLIB,              
//         DISP=(NEW,CATLG),                  
//         UNIT=SYSALLDA,                     
//         SPACE=(TRK,(20,5,10)),             
//         DCB=(RECFM=FB,LRECL=256,BLKSIZE=0),
//         DSNTYPE=LIBRARY                    
//*                                           
```

### 5.5 IZPCPYML

This job just copies DSN=IZP.SIZPPARM(IZPYAML) to IZP.PARMLIB(ZWEYAML)

```
//YAML2MVS EXEC PGM=IKJEFT01                                   
//*                                                            
//* Replace {components.izp.dataset.parmlib} with your desired 
//* PARMLIB location.                                          
//* Replace {components.izp.dataset.runtimeHlq} with the HLQ   
//* at which the SMP/E components were installed.              
//*                                                            
//IN       DD DSN=IZP.SIZPPARM(IZPYAML),                       
//            DISP=SHR                                         
//OUT      DD DSN=IZP.PARMLIB(ZWEYAML),                        
//            DISP=SHR                                         
//SYSTSPRT DD SYSOUT=*                                         
//SYSTSIN  DD *                                                
OCOPY INDD(IN) OUTDD(OUT) TEXT                                 
/*                                                             
```

### 5.6 Edit ZWEYAML

This is the crux of the deployment.

The original ZWEYAML sample file, with comments, can be viewed [here](https://github.com/zeditor01/zowetools/blob/main/code/ORIGYAML.TXT)

My ZWEYAML file can be viewed [here](https://github.com/zeditor01/zowetools/blob/main/code/ZWEYAML.TXT)



### 5.7 IZPGENER

This job generates a critical environment definition file and the customisation JCLS

It generates IZP.ENVIRON
```
IZP_ZOWE_RUNTIME="/usr/lpp/zwe/zwe217"                                                                                         
IZP_SCHEMA_CHAIN="/usr/lpp/IBM/izp/v1r2m0/bin/ums/izp-schema.json:/usr/lpp/zwe/zwe217/schemas/zowe-yaml-schema.json:/usr/lpp/zwe/zwe217/schemas/server-common.json"
IZP_CONFIG_CHAIN="PARMLIB(IZP.PARMLIB):FILE(/usr/lpp/zwe/zwe217/zowe.yaml)"
```                                          

It also generates the customisation JCLs in IZP.JCLLIB 
```
IZPA1   
IZPA1V  
IZPA2   
IZPA2V  
IZPA3   
IZPA3V  
IZPB0R  
IZPB0VR 
IZPB1R  
IZPB1VR 
IZPB2R  
IZPB2VR 
IZPB3R  
IZPB3VR 
IZPB4R  
IZPB4VR 
IZPC1R  
IZPC1VR 
IZPC2R  
IZPC2VR 
IZPD1R  
IZPD1VR 
IZPD2R  
IZPD2VR 
IZPD3R  
IZPD3VR 
IZPD4R  
IZPD4VR 
IZPD5R  
IZPD5VR 
IZPD6R  
IZPD6VR 
IZPD7R  
IZPD7VR 
IZPEXPIN
IZPIPLUG
IZPSTEPL
IZPUSRMD
```

### 5.8 IZPA3

IZPA1 and IZPA2 are not needed if useSAFOnly=true. (They allocate TEAMLIST and USERLIST datasets, which are no longer used if we always use SAF for authentication and authorisation controls.

Run IZPA3, which allocates a dataset (IZP.DBA.ENCRYPT) to dba encryption data

```
//IZPA3    EXEC PGM=IEFBR14                          
//ALLOC    DD   DSN=IZP.DBA.ENCRYPT,                 
//         DISP=(NEW,CATLG),                         
//         UNIT=SYSALLDA,                            
//         SPACE=(TRK,(20,5)),                       
//         DCB=(DSORG=PS,RECFM=FB,LRECL=80,BLKSIZE=0)
```


### 5.9 Execute Selected  ESM JCLs

Having read the notes, I only need to run a subset of the jobs. (I only need to run the RACF-jobs. And I am using usSAFOnly=true which means some other jobs are not needed ).

I Executed the following ESM JCLs

IZPB1R
IZPB2R 
IZPD1R 
IZPD2R ( this one did a PERMIT to the ZWESLSTC which is not an ID. So I modified it to do the PERMIT to the ZWESVUSR )
IZPD3R
IZPD4R ( I needed to modify this one, because it didnt work )
IZPD5R
IZPD7R 
IZPSTEPL




### 5.10 Encrypt DBA credentials

Run ```izp-encrypt-dba.sh```

Open a terminal session, and run this script, supplying the HLQ as an input parameter, to encrypt the token.

```
IBMUSER:/Z31A/usr/lpp/IBM/izp/v1r2m0/bin/ums/opt/bin: >./izp-encrypt-dba.sh IZP
IZPPI0079I - Start of izp-encrypt-dba.sh
IZP Credential Encryption Utility
Using PKCS #11 token label: IZPTOK
Using path to PKCS #11 library file: /usr/lpp/pkcs11/lib/csnpca64.so
Using database administrator username: IZPDBA1
Enter the password for the database administrator:

Reenter the password:

Supplied Credentials encrypted
IZPPI0080I - End of izp-encrypt-dba.sh. Return code 0
```

Take a peek in IZP.DBA.ENCRYPT for fun

```
********************************* Top of Data **********************************
/..<.î:ÄÑ.ËÄç ÎÄåÈ¦Ä:áÌ<.ÌøßÑ.¦Ä..Ïß.á.+ä.:ÂÏ...................................
Â..ëîøéîá.<.....................................................................
Ä....ÂÑ>/Ê`.@...................................................................
....Ñ ãäÏÀ..åÑìêê_ÈñÀÉ: ........................................................
À....ÂÑ>/Ê`.@...................................................................
..(ÏîÃã|...¦%.ê.`ñáÏ<.:é........................................................
******************************** Bottom of Data ********************************
```

### 5.11 IZPIPLUG

Run IZPIPLUG job

```
 ------------------------------------------------------------------------------------------------------------------------------
 SDSF OUTPUT DISPLAY IZPCUST1 JOB00983  DSID   102 LINE 0       COLS 02- 128                                                   
 COMMAND INPUT ===>                                            SCROLL ===> CSR                                                 
********************************* TOP OF DATA ******************************************************************************** 
IZPPI0079I - Start of izp-install-plugins.sh.                                                                                  
/tmp/.zweenv-1201/zwe-parmlib-3724 -> IZP.PARMLIB(ZWEYAML): text                                                               
Temporary directory '/tmp/.zweenv-1201' created.                                                                               
Zowe will remove it on success, but if zwe exits with a non-zero code manual cleanup would be needed.                          
bos extend currSize=0x0 dataSize=0x1836 chunk=0x1000 extend=0x1836                                                             
Installing file or folder=/usr/lpp/IBM/izp/v1r2m0/bin                                                                          
Install bin                                                                                                                    
Process ums/opt/bin/izp-install.sh defined in manifest commands.install:                                                       
2024-07-21 01:36:40 <ZWELS:67175104> IBMUSER INFO (zwe-components-install-process-hook) - commands.install output from izp is: 
                                                                                                                               
Successfully installed ZIS plugin: zos-newton-db2ifi                                                                           
Successfully installed ZIS plugin: zos-newton-discovery                                                                        
Successfully installed ZIS plugin: zos-newton-registry                                                                         
Successfully installed ZIS plugin: zos-newton-security                                                                         
Successfully installed ZIS plugin: zss-data-provider                                                                           
Successfully installed ZIS plugin: cidb                                                                                        
Successfully installed ZIS plugin: ums-security                                                                                
Successfully installed ZIS plugin: zos-newton-daj                                                                              
Successfully installed ZIS plugin: hlv-discovery                                                                               
- update zowe config /tmp/.zweenv-1201/.zowe-merged.yaml, key: "components.izp.enabled" with value: true                       
  * Success                                                                                                                    
Writing temp file for PARMLIB update. Command= cp -v "/tmp/.zweenv-1201/zwe-parmlib-3724" "//'IZP.PARMLIB(ZWEYAML)'"           
bos extend currSize=0x0 dataSize=0x1836 chunk=0x1000 extend=0x1836                                                             
IZPPI0080I - End of izp-install-plugins.sh. Return code 0                                                                      
******************************** BOTTOM OF DATA ******************************************************************************
```
    

### 5.12 IZPEXPIN

Submit IZPEXPIN to install any defined experiences. ( In this case, Db2 Admin Experience ).

```
 SDSF OUTPUT DISPLAY IZPCUST1 JOB00989  DSID   102 LINE 0       COLS 02- 128                                                   
 COMMAND INPUT ===>                                            SCROLL ===> CSR                                                 
********************************* TOP OF DATA ******************************************************************************** 
IZPPI0079I - Start of izp-cp-exp.sh                                                                                            
IZPPI0121I - Updating /tmp/izp-merge-16843471 with new elements from /usr/lpp/IBM/afx/v1r2m0/bin/admin-fdn-db2/var/conf/configu
IZPPI0031I - File copy status: OK - IZPDB2PM                                                                                   
IZPPI0121I - Updating /tmp/izp-merge-16843471 with new elements from /usr/lpp/IBM/afx/v1r2m0/bin/admin-fdn-db2/var/conf/configu
IZPPI0031I - File copy status: OK - IZPDAFPM                                                                                   
IZPPI0049I - Experience post-installation status: /usr/lpp/IBM/afx/v1r2m0/bin/admin-fdn-db2 OK                                 
alloc da('IZP.SAFXDBRM') dsorg(po) dsntype(library) tracks space(10,5) lrecl(80) blksize(0) recfm(f,b) new catalog             
alloc da('IZP.SAFXLLIB') dsorg(po) dsntype(library) tracks space(100,10) lrecl(0) blksize(32760) recfm(u) new catalog          
alloc da('IZP.SAFXSAMP') dsorg(po) dsntype(library) tracks space(10,5) lrecl(80) blksize(0) recfm(f,b) new catalog             
IZPPI0049I - Experience post-installation status: /usr/lpp/IBM/afx/v1r2m0/bin/admin-fdn-db2 OK                                 
IZPPI0080I - End of izp-cp-exp.sh. Return code 0                                                                               
******************************** BOTTOM OF DATA ****************************************************************************** 
```

The output above was from an earlier attempt, where I did try to install DB2 Admin Foundation. My latest attempt only tried to install UMS. Same difference.

### 5.13 Edit ZOWE STC JCL

The Started Task JCL must identify both the UMS ZWEYAML and the ZOWE zowe.yaml

```
//ZWESLSTC  PROC RGN=0M,HAINST='__ha_instance_id__'                    
//********************************************************************/
//* This program and the accompanying materials are                  */
//* made available under the terms of the                            */
//* Eclipse Public License v2.0                                      */
//* which accompanies this distribution, and is available at         */
//* https://www.eclipse.org/legal/epl-v20.html                       */
//*                                                                  */
//* SPDX-License-Identifier: EPL-2.0                                 */
//*                                                                  */
//* Copyright Contributors to the Zowe Project.                      */
//********************************************************************/
//*                                                                  */
//* ZOWE LAUNCHER PROCEDURE                                          */
//*                                                                  */
//* NOTE: this procedure is a template, you will need to modify      */
//*       #zowe_yaml variable to point to your Zowe YAML config      */
//*       file.                                                      */
//*                                                                  */
//* Check https://docs.zowe.org for more details.                    */
//*                                                                  */
//********************************************************************/
//ZWELNCH  EXEC PGM=ZWELNCH,REGION=&RGN,TIME=NOLIMIT,                  
// PARM='ENVAR(_CEE_ENVFILE=DD:STDENV),POSIX(ON)/&HAINST.'             
//STEPLIB  DD   DSNAME=ZWE217.SZWEAUTH,                                
//             DISP=SHR                                                
//SYSIN    DD  DUMMY                                                   
//SYSPRINT DD  SYSOUT=*,LRECL=1600                                     
//SYSERR   DD  SYSOUT=*                                                
//********************************************************************/
//*                                                                    
//* CONFIG= can be either a single path ex.                            
//*   CONFIG=/my/zowe.yaml                                             
//*                                                                    
//* Or a list of FILE() or PARMLIB() and colon : separated paths       
//*   in the form of                                                   
//*                                                                    
//*    +------------ : ------------+                                   
//*    V                           |                                   
//* >--+--FILE(ussPath)------------+--><                               
//*    |                           |                                   
//*    +--PARMLIB(dsname(member))--+                                   
//*                                                                    
//* Example:                                                           
//*   CONFIG=FILE(/my/long/path/to/1.yaml)\                            
//*   :PARMLIB(ZOWE.PARMLIB(YAML))                                     
//*                                                                    
//* In the above case, the \ is used as a line continuation.           
//*                                                                    
//* When using a list, files on left override properties               
//* from files to their right.                                         
//* Typically the right-most file should be the Zowe default yaml.     
//*                                                                    
//* Note: All PARMLIB() entries must all have the same member name.    
//*                                                                    
//********************************************************************/
//STDENV   DD  *                                                       
_CEE_ENVFILE_CONTINUATION=\                                            
_CEE_RUNOPTS=HEAPPOOLS(OFF),HEAPPOOLS64(OFF)                           
_EDC_UMASK_DFLT=0002                                                   
CONFIG=PARMLIB(IZP.PARMLIB(ZWEYAML))\                                  
:FILE(/usr/lpp/zwe/zwe217/zowe.yaml)                                   
/*                                                                     
```

### 5.14 Start ZSS

Enter z/OS Console command  ```S ZWESISTC,REUSASID=YES```  

### 5.15 Start ZOWE

Enter z/OS Console command ```S ZWESLSTC```

## 6. Validate ZOWE with UMS

The errors I get when I connect to ZOWE are

1. Session Timeout
2. 401 error in the Unified Experience App


I will review the ZOWE started task output and the syslog.



