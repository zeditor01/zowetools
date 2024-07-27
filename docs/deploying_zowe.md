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

This matches the file structure described by the Zowe documentation

![zowe_directory](/images/zowedir.JPG)

### 4.2 Client Side components

Not covered at this point, because my Db2 tools only need the Server Side component. I will update this guide later when I deploy the CLient Side components.

## 5. Preparing the zowe.yaml file of parameters

The entire Zowe deployment is defined by the zowe.yaml file. This configuration file is used by
1. the ZWE INSTALL script to create the ZOWE installation datasets
2. the ZWE INIT script to customise the ZOWE instance
3. at Zowe runtime

If you've not come across yaml files yet, yaml stands for 'yet another markup language'. The zowe.yaml file is documented [here](https://docs.zowe.org/stable/appendix/zowe-yaml-configuration/). It has 6 sections

1. zowe (Defines global configurations specific to Zowe, including default values)
2. java (Defines Java configurations used by Zowe components)
3. node (Defines node.js configurations used by Zowe components)
4. zOSMF (Tells Zowe your z/OSMF configurations)
5. components (Defines detailed configurations for each Zowe component or extension)
6. haInstances (Defines customized configurations for each High Availability (HA) instance)

I prepared the following [zowe.yaml](https://github.com/zeditor01/zowetools/blob/main/code/zowe.yaml) file for my system. ***Hint: right mouse click + open link in new tab.***

Note the following parameters

setup-dataset section
* line 43 provides the HLQ where the ZOWE datasets will be installed later on using the 'ZWE INSTALL' script
* line 46 specifies your chosen proclib where the startup procedures will be created
* line 49 specifies your chose parmlib for plugins
* line 56 specifies the PDS to create the customisation JCL members in
* line 58 specifies the loadlib from the chosen installation method (convenience build in my case)
* line 60 specifies the APF-authorised loadlib from the chosen installation method (convenience build in my case)
* line 63 specifies the APF authorized LOADLIB for Zowe ZIS Plugins from the chosen installation method (convenience build in my case)

setup-security section (needs to be filled in to use RACF and RACF keyrings)
* line 69 specifies that we will use RACF security. This is an opensource product, and prepares security configurations for multiple security products)
* line 71 - 77 specifies the RACF groups to create and use (I used ZWEADMIN for all three groups)
* lines 81 and 83 specifies the user for the main Zowe task (ZWESVUSR) and for the ZIS server (ZWESIUSR)
* lines 85 - 91 specifies the started task names ZWESLSTC (main server) ZWESISTC (zis server) and ZWESASTC (zis auxilliary server)

certificate section. The example zowe.yaml file provides templates for 5 different certificate configurations. I chose scenario 3 - Zowe generated z/OS Keyring with Zowe generated certificates.
* line 180 specifies that I wish to use a RACK keyring to hold certificates
* line 185 specifies the Keyring name (ZoweKeyring)
* line 188 specifies the label of my certificate
* line 190 specifies the label of my CA certificate
* line 193 - 200 specifies options distinguished name values
* line 202 ensures that the certificate won't expire until well after i retire
* line 208 and 210 specify the hostname (s0w1.dal-ebis.ihost.com) and IP address (192.168.1.171) for my z/OS system

cacheing service section
line 259 - 256 specifies to use Non-RLS VSAM for the cacheing service

runtime directories
* line 282 specifies the runtimeDirectory: "/usr/lpp/zwe/zwe217"
* line 286 specifies the logDirectory: /global/zowe/logs
* line 290 specifies the workspaceDirectory: /global/zowe/workspace
* line 294 specifies the extensions directory, where plugins will be linked: /global/zowe/extensions

ports
* line 364 specifies the port that Zowe listens for browser connections on

generated certificates: Leave this section blank. It will be updated during the 'ZWE INIT' Script
* line 396 - 420 specify the RACF keystore and truststore to be used for certificates

java
* line 447 specifies the path to Java 8. Note that Zowe V2.18 depends on Java 8 for the install, but can run with current Java 17.

nodeJS
* line 462 specifies the path to nodeJS

z/OSMF
* lines 471 - 477 specify the connection details for z/OSMF
  
I left everything else from the example zowe.yaml file to default.

Be careful of case sensitive parameters (paths and certificate details)

Once you've edited the zowe.yaml file, everything else should flow easily.


## 6. Running the 'ZWE INSTALL' Script

The zowe.yaml file controls the ZWE INSTALL Script.

First, Check your USS environment variables. You may edit your .profile to include the following specifications

```
# JAVA                                                                                               
export JAVA_HOME=/usr/lpp/java/J8.0_64                                 
export PATH=$JAVA_HOME/bin:$PATH                                                                          
# Node.js                                                              
export NODE_HOME=/usr/lpp/IBM/cnj/v20r11                               
export PATH=/usr/lpp/IBM/cnj/v20r11/bin:$PATH                          
export PATH=/usr/lpp/zwe/zwe217/bin:$PATH                              
```

Open a new USS shell (in order to execute your .profile) and run the following command.

```
zwe install -v -c /usr/lpp/zwe/zowe/zowe.yaml
```

It should chunter away and create the following z/OS datasets
```
'ZWE217.SZWEAUTH'                              A3USR5
'ZWE217.SZWEEXEC'                              A3USR2
'ZWE217.SZWELOAD'                              A3USR6
'ZWE217.SZWESAMP'                              A3USR7 
```

That's it : The execution libraries are in place.

## 7. Running the 'ZWE INIT' Script

You can run the commands individually - but the ```--update-config --allow-overwrite``` options don't work so well like that.

```
zwe init mvs --update-config --allow-overwrite -v -c /usr/lpp/zwe/zowe/zowe.yaml
zwe init security --update-config --allow-overwrite -v -c /usr/lpp/zwe/zowe/zowe.yaml
zwe init apfauth --update-config --allow-overwrite -v -c /usr/lpp/zwe/zowe/zowe.yaml
zwe init certificate --security-dry-run -v -c /usr/lpp/zwe/zowe/zowe.yaml
zwe init certificate --update-config --allow-overwrite -v -c /usr/lpp/zwe/zowe/zowe.yaml
zwe init vsam --update-config --allow-overwrite -v -c /usr/lpp/zwe/zowe/zowe.yaml
zwe init stc --update-config --allow-overwrite -v -c /usr/lpp/zwe/zowe/zowe.yaml
```

So - we did the whole schmoogle in one step. My experience was that this worked first time.

```
zwe init --update-config --allow-overwrite -v -c /usr/lpp/zwe/zowe/zowe.yaml
```

You might want to check some of the artefacts that were created. 
For example, a RACF query ```racdcert id(ZWESVUSR) listring(ZoweKeyring)``` to see the contents of the ZoweKeyring that was created.

```
Digital ring information for user ZWESVUSR:                           
                                                                      
  Ring:                                                               
       >ZoweKeyring<                                                  
  Certificate Label Name             Cert Owner     USAGE      DEFAULT
  --------------------------------   ------------   --------   -------
  zowes0w1ca                         CERTAUTH       CERTAUTH     NO   
                                                                      
  zowes0w1                           ID(ZWESVUSR)   PERSONAL     YES  
                                                                      
  zOSMFCA                            CERTAUTH       CERTAUTH     NO   
```

Under certificate model 3, Zowe created a self-signed certificate and CA Cert, and placed it in ZoweKeyring, alongside the zOSMF CA Cert.

You should also check your PROCLIB to see the members placed in it.

```
ZWESASTC
ZWESISTC
ZWESLSTC
```


## 8. Starting ZOWE

A few pieces of housekeeping before starting Zowe

### 8.1 Edit zowe.yaml again

Need to edit the zowe.yaml file: Need to comment out ```createZosmfTrust: true``` for zowe config validate and zowe start

### 8.2 Edit Program Properties Table for these tasks

Edit USER.Z31A.PARMLIB(SCHEDAL)
```
PPT PGMNAME(ZWESIS01) NOSWAP KEY(4) /* Zowe cross memory           */
PPT PGMNAME(ZWESAUX)  NOSWAP KEY(4) /* Zowe auxiliary (AUX)        */
```

### 8.3 Start ZWESISTC

z/OS operator console command: ```S ZWESISTC,REUSASID=YES```

### 8.4 Start ZWESLSTC

z/OS operator console command: ```S ZWESLSTC```



## 9. Logging on to ZOWE

To get a secure browser connection to ZOWE you will need to import the CA certificates into your PC or browser.

Export the two CA certificates to files.

View them (for example ZOSMFCA below).

```
  File  Edit  Edit_Settings  Menu  Utilities  Compilers  Test  Help                     
--------------------------------------------------------------------------------------- 
-DSC- VIEW IBMUSER.ZOSMFCA.CRT                                     Columns 00001 00080  
Command ===>                                                          Scroll ===> CSR   
****** ********************************* Top of Data ********************************** 
==MSG> -Warning- The UNDO command is not available until you change                     
==MSG>           your edit profile using the command RECOVERY ON.                       
000001 -----BEGIN CERTIFICATE-----                                                      
000002 MIIDfDCCAmSgAwIBAgIBADANBgkqhkiG9w0BAQsFADA9MQwwCgYDVQQKEwNJQk0x                 
000003 EDAOBgNVBAsTB0laVURGTFQxGzAZBgNVBAMTEnpPU01GIHozMWEgUm9vdCBDQTAg                 
000004 Fw0yNDA0MDkxMzAwMDBaGA8yMDUwMDEwMjEyNTk1OVowPTEMMAoGA1UEChMDSUJN                 
000005 MRAwDgYDVQQLEwdJWlVERkxUMRswGQYDVQQDExJ6T1NNRiB6MzFhIFJvb3QgQ0Ew                 
000006 ggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQDI5+OZAmiBv3EbBr9YMrs4                 
000007 sApDybYv4+ibxqggs9iGIL94LTX2L9UNDJsXzTAS7EKHzuA/DpmFVFbWIqqnSvFB                 
000008 BmFRK1WoQQTVb+Fpg8aV9LxXRn53zDIl2nhLQ+iPfzfAD19eLSzeXiOFyaOrTXRJ                 
000009 ARs2rIzHNx8qIzt7rrYazuG8xW1CpWA4DlGnmA8maAOdDG+zEtivHspkrKkgnS9f                 
000010 KFkTzSo8rcJuiryGi8JeRIK2CJeESNC8p2ZGjySknHnSI+cFkOAA7EgyfmKZY4DY                 
000011 NJEAnKHidzCamYPC+IwRIABp/Nkmvnwu4Iw6Wk0vOfDd00gDsQPV7ggRrPUBRyiH                 
000012 AgMBAAGjgYQwgYEwPwYJYIZIAYb4QgENBDIWMEdlbmVyYXRlZCBieSB0aGUgU2Vj                 
000013 dXJpdHkgU2VydmVyIGZvciB6L09TIChSQUNGKTAOBgNVHQ8BAf8EBAMCAQYwDwYD                 
000014 VR0TAQH/BAUwAwEB/zAdBgNVHQ4EFgQUNJc5WplDrPjC3SO4yftFW+XdNgQwDQYJ                 
000015 KoZIhvcNAQELBQADggEBAATIXjB+UomoMMyDTF6HO8cRrYPLAoxKRHKtYqDzuwWj                 
000016 aXkAPFRh7JiTzLJ2RuWdqBSGw4cuYsTYhOZqvQ2kPGayHjQcAv8zNJCK4pNuKs/M                 
000017 Kf7UwtgXtcgqFH/9hNlmXa3RwATqwtqFY6F70GoLvb3DPj6XL4JrOKCZRFZu7CvH                 
000018 bok2uG3svXl/2upCyk28sc36dlZuI92QHIdKGqJrQT6TUXP6EBMXwy0MUYYqtTaz                 
000019 mXfd+A9KPO4g2x2tmdxEsnH8PUKGE3e9Z1TE8hxfWpdZEjCyb5NKNohxRP2dpF+N                 
000020 Zgidu4iXMqSP6yjNgUK204kp3QkC6V8v/a9UGt4XvLg=                                     
000021 -----END CERTIFICATE-----                                                        
****** ******************************** Bottom of Data ******************************** 
```

Copy them to and paste the contents to PC files

Import them to the Local root CA truststore on your PC

Now, Open a Browser Window to logon to ZOWE

```
https://s0w1.dal-ebis.ihost.com:7554/zlux/ui/v1
```

![zowe_logon](/images/zowe7556.JPG)


## 10. Using ZOWE

ZOWE has a number of standard utilities, includind

* JES Explorer
* Dataset Browser
* 3270 emulator

![zowe_services](/images/zowe7554.JPG)

These will be augmented as we install additional plugins

* Db2 Administration Foundation
* Db2 Automation Expert
* and more

## 11. ZOWE Automation

Place the following commands in your startup and shutdown PARMLIB members

```
USER.Z31A.PARMLIB(VTAMALL)

S ZWESISTC,REUSASID=YES 
S ZWESLSTC
```

```
USER.Z31A.PARMLIB(SHUTALL)

P ZWESISTC 
P ZWESLSTC
```


