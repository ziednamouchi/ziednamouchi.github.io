---
layout: default
---

# Hackzone: RexR3xRex Walkthrough

![hz](public/img/hz.png)



Presentation
------------

During an Active Directory pentest mission, i successfully pwned a user password from this network traffic capture, can you reproduce what I have done and submit the flag in this format : HZVIII{user@domain:password}

Vulnerability
-------------

Kerberos Preauthentication: cracking user password from AS-REQ or AS-REP.
Actually Pre-authentication is the first step first in Kerberos authentication. 

How Preauth works? When requesting a TGT a user encrypts the timestamp with its own password and sends [the PA-DATA](https://tools.ietf.org/html/rfc4120#page-60) (it is part of the AS-REQ). The KDC will attempt to decrypt it and validate that the right password was used and that it is not replaying a previous request.  From there, the TGT will be issued for the user to use for future authentication.

Luckily, preauthentication is required by default in Active Directory.  However, when capturing the network an offline bruteforce of the password is possible.


Write up
-----------

- Analyse the given pcapng and find the kerberos traffic
![1](public/img/1.png)

- Try to understand how the kerberos authentication works
- Find this information in the pcapng: username (CNameString: kerbdog), domain (realm: sbb.local), encryption type (etype: AES '18') and the cipher.
![2](public/img/2.png)

Ps: The cipher can be from the AS-REQ or AS-REP. When cracked, they can both give the user's password.

- if using the cipher from AS-REQ. The cracking process will be more challenging. John does not crack this hash and neither do hashcat for e-type 18 (mode 19900). Luckily, [the beta version of hashcat](https://hashcat.net/beta/) supports that mode. 
![3](public/img/3.png)
According to hashcat documentation the hash must be like: $krb5pa$18$username$domain$cipher
> $krb5pa$18$kerbdog$sbb.local$sbb.localkerbdog$3863ce7fb6c523b9dad5a2b24aa437896216d
27f740060f87829a2003ac32d1dad3ec07e7874f403b1077e350571e64bb033e65529c7c962

![4](public/img/4.png)

if using the cipher from AS-REP (which is similar to AS-REP roasting) you can either use john with hash like: $krb5asrep$18$domainuser$cipher 
> $krb5asrep$18$SBB.LOCALkerbdog$921353c94ab565d7c95ddd4b32ba179e135ca82d19a80daac0ade3
c8e4976fc521681268cf6d77b181f73a50cbf1fc43669e5453bd76577d66068138b14b281bebbe4c81c57fd
21199b98de9352be82697de9b103a9639d70795208c2375105bef1a208e1da03d315aa69a9da3436c78d28c0
d3974dc6d84878c29c2d936b0f3370ee159419d58dbb4283f3a4f7300225ec48dceb57061e61977f108f279a
256d528b329bf9ad02cb7834c6cf2b7bc7eac243678d66d27e6bcc64057237bfae442fc8bc53f63b22ef7fcc
92c6d0358e20eed648de7aac48c988adff642b922885878cc75ae583c1ee34693ca1e16dbd6584fa05aca3b5f2
17a574567abac601c5aa3d76a40a3fe5245087f246476cfd9ef17eba8d9125a09e7d4da6e8f74876d17a7a1f5c
861944d45df61654c96fcb444dc17$2b9b90918c5bb823c4ffe0db

or hashcat (mode 19700)

Distribution
-------------
https://drive.google.com/open?id=1Yb_ipMIh1tCXytWApILBreIp5q8NXeML


Flag
----------

`HZVIII{kerbdog@sbb.local:Kerberos!99}`

References
----------

-https://www.harmj0y.net/blog/activedirectory/roasting-as-reps/
- https://ldapwiki.com/wiki/Kerberos%20Pre-Authentication

