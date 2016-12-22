# Client-Side Certificate Authentication

This document is intended to explain what is Client-Side Certificate Authentication and how to use it.

Disclaimer: I am not a security expert. There may be some errors or incomplete explanations and therefore I encourage you to correct and improve this document.

1. [Introduction](#1-introduction)
    * [What is Client-Side Certificate Authentification (CSCA)?](#-what-is-client-side-certificate-authentication-csca)
    * [How do this work?](#-how-do-this-work)
2. [Setting up a CSCA](#2-setting-up-a-csca)
    1. [Requirements](#i-requirements)
    2. [Setup the CA](#ii-setup-the-ca)
    3. [Generate a client certificate](#iii-generate-a-client-certificate)
    4. [Going further](#iv-going-further)
3. [Documentation](#3-documentation)

## 1) Introduction

### * What is Client-Side Certificate Authentication (CSCA)?

CSCA is a way to authenticate your clients to your server using SSL certificates.

The first advantage is security: Using SSL certificates to communicate between clients and server is the way to go to secure your communications.

The second advantage is to free yourself from a datastore with the common username/password schema. (Which often imply security threats with "p@ssword123" and others".)

### * How do this work?

Basically, you will establish yourself as a Certificate Autority (CA) to verify the certificate of a client and allow or deny access to your server ressource.

When you want to allow aclient access, you will issue a signed certificate that he will use in each of his requests to the server.

Every time your server receive a request, he will verify the given certificate with your CA's certificate.

## 2) Setting up a CSCA

First, you will setup the CA, then you will generate client certificates.

### i) Requirements

This document specify the procedure used on a linux OS with root access, but you can probably follow them with some tweaking on other OS.

You will need an up-to-date OpenSSL to ensure you have the latest security patches.

### ii) Setup the CA

OpenSSL comes with a default CA. We won't use it, but we will customize OpenSSL's config file to our needs.
```
# Probably at /etc/ssl/openssl.cnf
[ ca ]
default_ca = CA_default					# The name of the CA configuration to be used.
										# can be anything that makes sense to you.
										
[ CA_default ]
dir = /etc/ssl/ca 						# Directory where everything is kept
certs = $dir/certs 						# Directory where the issued certs are kept
crl_dir = $dir/crl 						# Directory where the issued crl are kept
database = $dir/index.txt 				# database index file.

new_certs_dir = $dir/certs 				# Default directory for new certs.
certificate = $dir/ca.crt 				# The CA certificate
serial = $dir/serial 					# The current serial number
crlnumber = $dir/crlnumber 				# The current crl number
										# must be commented out to leave a V1 CRL
crl = $dir/crl.pem 						# The current CRL
private_key = $dir/private/ca.key 		# The private key
RANDFILE = $dir/private/.rand 			# private random number file
x509_extensions = usr_cert 				# The extentions to add to the cert
name_opt = ca_default 					# Subject Name options
cert_opt = ca_default 					# Certificate field options
default_days = 365 						# how long to certify for
default_crl_days= 30 					# how long before next CRL
default_md = sha256						# use public key default MD
preserve = no 							# keep passed DN ordering
policy = policy_match
```
> Note: SHA1 will be depreciated on 01/01/2017. You should use either SHA256 or SHA512.
> More on this here: http://www.infoworld.com/article/3064654/security/tick-tock-time-is-running-out-to-move-from-sha-1-to-sha-2.html

You then need to create the files and folders we added to the config file.
```
mkdir -p /etc/ssl/ca/certs/users
mkdir /etc/ssl/ca/private
```

Alright, now comes the nice stuff! We will begin by generating a private key for our CA.
```
openssl genrsa -out /etc/ssl/ca/private/ca.key 4096
```

> Note: 4096 correspond to the number of bits in our key. You can also use a 2048 size.
> More on this here: https://danielpocock.com/rsa-key-sizes-2048-or-4096-bits

> Note: You can protect the private key with a password using the -des3 argument.
> More on this here: http://stackoverflow.com/a/21082690

Then, we will create our CA's certificate following the X.509 standard.
```
openssl req -new -x509 -key /etc/ssl/ca/private/ca.key -out /etc/ssl/ca/certs/ca.crt
```

You will be asked some information to define the "Distinguished Name" of the certificate.
The most important information to give is the "Common Name" field which is typically your domain name (e.g: api.mydomain.tld).

> Note: The certificate will be valid for a limit number of days. Either the default specified in the config file or the one given with the -days 365 argument.
> More on this here: http://security.stackexchange.com/a/85991

### iii) Generate a client certificate

Now that you have the CA's certificate, let's begin with the client certificates.

First, you need to generate a private key. (Same command as before)
```
openssl genrsa -out /etc/ssl/ca/certs/users/user001.key 4096
```

Now that you have the key, you will need to make a Certificate Signing Request. This is NOT the client's certificate.
```
openssl req -new -key /etc/ssl/ca/certs/users/user001.key -out /etc/ssl/ca/certs/users/user001.csr
```

> Note: You can create the .key and the .csr in one line and without prompt.
> More on this here: http://www.shellhacks.com/en/HowTo-Create-CSR-using-OpenSSL-Without-Prompt-Non-Interactive

Then this file will be used by the CA to generate the client's certificate.
```
openssl x509 -req -in /etc/ssl/ca/certs/users/user001.csr -CA /etc/ssl/ca/certs/ca.crt -CAkey /etc/ssl/ca/private/ca.key -CAserial /etc/ssl/ca/serial -CAcreateserial -out /etc/ssl/ca/certs/users/user001.crt
```

Well done! You've done the hard part.

You now have a client certificate which can be valided by your CA certificate.

Just to be sure, run this command to test the CA's certificate:
```
openssl verify /etc/ssl/ca/certs/ca.cert
```

You should expect something like that:
```
/etc/ssl/ca/certs/ca.crt: OK
```

> Note: If you get an error 18, run this command

> `cp /etc/ssl/ca/ca.crt /usr/local/share/ca-certificates/ && update-ca-certificates`

Now run this command to test the user's certificate:
```
openssl verify -CAfile /etc/ssl/ca/certs/ca.crt /etc/ssl/ca/certs/users/user001.crt
```

You should expect something like that:
```
/etc/ssl/ca/certs/users/user001.crt: OK
```

> Note: If you get an error 18, this mean that you used the same Common Name than your CA.
> More on this here: http://stackoverflow.com/a/19738223

### iv) Going further

Section in construction...
- CSCA on Apache
- CSCA on Nginx
- Certificate revoking
- Certificate renewal

## 3) Documentation

During my learning of CSCA, I found quite of a lot of documentation. I listed them here:

https://arcweb.co/securing-websites-nginx-and-client-side-certificate-authentication-linux/

https://gist.github.com/kyledrake/d7457a46a03d7408da31

https://developer.salesforce.com/blogs/developer-relations/2011/05/generating-valid-self-signed-certificates.html

http://nategood.com/client-side-certificate-authentication-in-ngi

http://unix.stackexchange.com/questions/103461/get-common-name-cn-from-ssl-certificate

http://mindref.blogspot.fr/2012/02/openssl-renew-certificate.html

https://www.win.tue.nl/hashclash/rogue-ca/
