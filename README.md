# Security-HW7
## Problem statement 
Design and build a PKI infrastructure that includes Root CA, Signing CA, and TLS Certificate,
E.g., as described here https://pki-tutorial.readthedocs.io/en/latest/simple 
Use the TLS certificate to install a web server, e.g. tomcat, https://tomcat.apache.org/tomcat-7.0-doc/ssl-howto.html

## Implementation

We assume an organization named “SJSU CMPE272”, controlling the domain “securityhw.cmpe272.org”. The organization runs a small PKI to secure its intranet traffic

With the modified https://pki-tutorial.readthedocs.io/en/latest/simple/root-ca.conf.html and https://pki-tutorial.readthedocs.io/en/latest/simple/signing-ca.conf.html files we implement this using below steps

### Step 1:
#### Creating root CA
##### Create directories

```

mkdir -p ca/root-ca/private ca/root-ca/db crl certs
chmod 700 ca/root-ca/private
```

##### Create root database
```
cp /dev/null ca/root-ca/db/root-ca.db
cp /dev/null ca/root-ca/db/root-ca.db.attr
echo 01 > ca/root-ca/db/root-ca.crt.srl
echo 01 > ca/root-ca/db/root-ca.crl.srl
```
##### Create root CA request
```
openssl req -new \
    -config etc/root-ca.conf \
    -out ca/root-ca.csr \
    -keyout ca/root-ca/private/root-ca.key
  ```
  ##### Create root CA certificate
```
openssl ca -selfsign \
    -config etc/root-ca.conf \
    -in ca/root-ca.csr \
    -out ca/root-ca.crt \
    -extensions root_ca_ext
  ```
  
 ### Step 2:
#### Creating signing CA
##### Create directories
```
mkdir -p ca/signing-ca/private ca/signing-ca/db crl certs
chmod 700 ca/signing-ca/private
```
##### Create signing database
```
cp /dev/null ca/signing-ca/db/signing-ca.db
cp /dev/null ca/signing-ca/db/signing-ca.db.attr
echo 01 > ca/signing-ca/db/signing-ca.crt.srl
echo 01 > ca/signing-ca/db/signing-ca.crl.srl
```
##### Create signing CA request
```
openssl req -new \
    -config etc/signing-ca.conf \
    -out ca/signing-ca.csr \
    -keyout ca/signing-ca/private/signing-ca.key
  ```
  

##### Create signing CA certificate
```
openssl ca \
    -config etc/root-ca.conf \
    -in ca/signing-ca.csr \
    -out ca/signing-ca.crt \
    -extensions signing_ca_ext
  ```
  
  
### Step 3:
#### Operate signing CA
##### Create TLS server request
```
SAN=DNS:securityhw.cmpe272.org \
openssl req -new \
    -config etc/server.conf \
    -out certs/securityhw.cmpe272.org.csr \
    -keyout certs/securityhw.cmpe272.org.key
 ```
##### Create TLS server certificate
```
openssl ca \
    -config etc/signing-ca.conf \
    -in certs/securityhw.cmpe272.org.csr \
    -out certs/securityhw.cmpe272.org.crt \
    -extensions server_ext
 ```


### Step 5:
 Install nginx webserver on ubuntu linux machine install the generated certificate from the previous step by making changes to configuration file as shown below

<img width="666" alt="14_Nginx_Sites-enabled" src="https://user-images.githubusercontent.com/85700971/201513724-5ed25960-fee8-4c6e-b763-21a0b5fea695.png">

### Step 6:

Issued TLS server cerificate to the nginx webserver at https://securityhw.cmpe272.org

<img width="872" alt="16_Website" src="https://user-images.githubusercontent.com/85700971/201513791-8e866422-ed8f-4556-9beb-36be5352e005.png">
