# How to connect ASDM to Cisco ASA

Hello everyone who reads this article. Dedicated to those who use the pirate cisco asa and / or those who have difficulties in connecting asdm and already about 100 times ask themselves the question - "how to make it work!???". Therefore, here is a detailed, step-by-step guide on how to connect cisco asa and asdm.
---
## My system configurations: 

asdm 7.1-15-100

Cisco Adaptive Security Appliance Software Version 9.1(5)16 

Host from which cisco asa will be controlled - Windows Server 2012 (just because) you can use any windows system win7 for example) 

Unetlab and after testing on gns3 

### ! Java 6 version! asdm not work properly with latest java environment

## 1. Let`s Begin 
Create interfaces (you must know how do it, i just leave here my configuring inerfaces)

    interface Ethernet0
  
    duplex full
  
    nameif inside
  
    security-level 100
  
    ip address 192.168.1.1 255.255.255.0

    interface Ethernet1
 
    duplex full
  
    nameif outside
  
    security-level 0
  
    ip address dhcp

---
## 2. Allow icmp

ASA in some cases does not ping your server and at the same time during a ping it gives you ????? which means that icmp packets unknown 

So need to allow icmp, ip, tcp with acl:

        access-list allow_icmp extended permit icmp any any
        access-list allow_icmp extended permit ip any any
        access-list allow_icmp extended permit tcp any any

After which we apply the acl to the interfaces:

        access-group allow_icmp in interface inside
        access-group allow_icmp out interface inside

(after you get access to asdm, you can safely change these rules by making them more stringent, for now we are permit everything so that no problems arise)
---
## 3. Create user for logining and communicate with asdm
        username test password 123 privilege 15>
Then allow authorization and authentication otherwise asa, even with the correct data, will not let you inside:

        aaa authentication http console LOCAL
        aaa authentication enable console LOCAL
        aaa authentication ssh console LOCAL
        aaa authentication serial console LOCAL
        aaa authorization command LOCAL 
---
## 4. Enable http server and preparing asdm image

        copy ftp://hope:dust@178.90.125.15/asdm-715-100.bin flash:

        asdm image flash:asdm-715-100.bin - explicitly indicate the image waht we downloaded from ftp server

        http server enable

        http 192.168.1.0 255.255.255.0 inside - specify FROM which network will be allowed access to which interface 
---
## 5. Configuring host machine 

You need to download java 6 version with asdm 100% work properly. 

Install java environment and now we ready to try. 
---
## 6. Cherished moment 

Open your browser and type your cisco asa address: (https required!)

https://asa_ip/admin

Then you are redirected to 

https://asa_ip/admin/public/index.html

Were you just click on "Install ASDM launcher" after downloading install it and run. 
!Warning! Before that you should to download asa certificate what generated by asa using your browser's certificate wizard. Probably, java will not allow you to connect via https using a self-written certificate that was generated by asa, so go to Control Panel search for Java and click on it. Go to security --> Certificates --> import (check all files) and select the certificate that is generated by your asa. 

Thus, java will trust this site and you will not have errors, like this:

        javax.net.ssl.SSLHandshakeException: java.security.cert.CertificateException: Java couldn't trust Server 

Enter your asa ip and credentials for user what you created on asa in asdm console and click "ok"

#### Well-done! Now you have an access to asdm web interface for your asa!) 
---
IF you still haven't access the management console and you get the error "unable connect to asa_ip" This means that either:
1. You haven't installed the correct version of java 
2. You did not allow connections and / or you did not set authentication and authorization using the local database (point 3) 
3. You entered invalid credentials

