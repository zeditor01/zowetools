[home](https://github.com/zeditor01/zowetools)

# Deploying ZOWE 

This document is an audit trail of deploying ZOWE V2.17

 ***Contents***

1. Links to ZOWE Documentation
2. Pre-Requisite Software
3. Planning the Deployment
4. Installing the ZOWE product
5. Running the 'ZWE INSTALL' Script
6. Running the 'ZWE INIT' Script
7. Starting ZOWE
8. Logging on to ZOWE
9. Using ZOWE
10. ZOWE Automation


## 1. Links to ZOWE Documentation

[zowe project home](https://www.zowe.org/)

[zowe documentation](https://docs.zowe.org/)

[zowe downloads](https://www.zowe.org/download)






Verify CIM
```
D A,CFZCIM

RESPONSE=S0W1                                                            
 CNZ4106I 09.36.14 DISPLAY ACTIVITY 883                                  
  JOBS     M/S    TS USERS    SYSAS    INITS   ACTIVE/MAX VTAM     OAS   
 00020    00042    00001      00037    00031    00001/00040       00052  
 CFZCIM NOT FOUND    

S CFZCIM

RESPONSE=S0W1                                                            
 CNZ4106I 12.59.38 DISPLAY ACTIVITY 931                                  
  JOBS     M/S    TS USERS    SYSAS    INITS   ACTIVE/MAX VTAM     OAS   
 00020    00043    00001      00037    00029    00001/00040       00051  
  CFZCIM   CFZCIM   *OMVSEX  OWT  SO  A=004F   PER=NO   SMC=000          
                                      PGN=N/A  DMN=N/A  AFF=NONE         
                                      CT=001.575S  ET=00.50.01           
                                      WUID=STC09875 USERID=CFZSRV        
                                      WKL=STARTED  SCL=STCLOM   P=1      
                                      RGP=N/A      SRVR=NO  QSC=NO       
                                      ADDR SPACE ASTE=7EEB63C0           

Edited USER.Z31A.PARMLIB(VTAMALL) to start CMI
```

Hell - Lets not review all the other z/OSMF setup stuff. Lets just find out if it works by deploying ZOWE


## Navigating the ZOWE Documentation

It's a bit all over the place, and esy to get lost. First time around I read the convenience build install instructions and pressed "next" to be passed on to the container install. WTF!

You need to read everything very slowly and carefully to spot the hidden pointers to the next logical step for your chosen installation path.

[Step 1 - check the Convenience Build installation docco here](https://docs.zowe.org/stable/user-guide/install-zowe-zos-convenience-build)

[Step 2 - then follow the initialise zos steps here](https://docs.zowe.org/stable/user-guide/initialize-zos-system)


 
## 2. Pre-Requisite Software

You must ensure that some prequisite software is installed on z/OS before you install zowe. The main System Requirements are listed here
* AXR (System REXX)
* CEA (Common Event Adapter - of z/OSMF)
* CIM (Common Information Model - of z/OSMF)
* CONSOLE and CONSPROF commands must exist in the authorised command table
* Java (V8 or later)
* NodeJS
* TSO Region Size - minimum 65,536 KB
* Userids - OMVS Segment

Please refer to the current list of pre-requisites [here](https://docs.zowe.org/stable/user-guide/zosmf-install)  before you start.

## 3. Planning the Deployment


## 4. Installing the ZOWE product


## 5. Running the 'ZWE INSTALL' Script


## 6. Running the 'ZWE INIT' Script


## 7. Starting ZOWE


## 8. Logging on to ZOWE


## 9. Using ZOWE


## 10. ZOWE Automation




