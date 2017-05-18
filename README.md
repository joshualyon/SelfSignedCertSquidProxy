# How To: Generate a Self Signed Certificate
This document describes how to generate a self signed SSL certificate which is **compatible with Chrome 58+** where the `net::ERR_CERT_COMMON_NAME_INVALID` error is shown with certificates that are generated by following the OpenSSL CLI prompts. 

In particular, these certificates will work with **SquidProxy** for creating an **HTTPS enabled proxy** that will securely proxy your communication to the internet, but will also work with web servers or other secure network communication where certificates are used and a SAN is required.

1. Generate the certificate configuration file: create a new file called `certConfig.txt` with the contents below. We'll use this file to fill in the required specifications and feed this file into OpenSSL to generate the actual certificate.
    ```
    [req] 
    distinguished_name = req_distinguished_name 
    x509_extensions = v3_req 
    prompt = no 
    [req_distinguished_name] 
    C = US 
    ST = TX 
    L = Irving 
    O = MyCompany 
    OU = MyGroup 
    CN = my.domain.com 
    [v3_req] 
    keyUsage = keyEncipherment, dataEncipherment 
    extendedKeyUsage = serverAuth 
    subjectAltName = @alt_names 
    [alt_names] 
    DNS.1 = my.domain.com 
    ```
1. Generate the PEM file
    ```
    openssl req -x509 -nodes -days 1365 -newkey rsa:2048 -keyout ./cacert.pem -out ./cacert.pem -config ./certConfig.txt
    ```
1. Generate the CER file to import into the Windows "Trusted Root Certification Authorites" store
    ```
    openssl x509 -inform PEM -in cacert.pem -outform DER -out certificate.cer
    ```
1. Import the self-signed CER file into the Windows Trusted Root CA Store:
   1. Double click the CER file on the client machine to open it
   1. Click `Install Certificate...`
   1. Select the `Local Machine` option and click **Next**  
      *(Accept the elevated priviledges request if prompted)*
   1. Choose `Place all certificates in the following store`
   1. Click `Browse` choose `Trusted Root Certification Authorites` and click **OK**
   1. Click **Next** and then **Finish**

## SquidProxy Configuration
1. Configure SquidProxy to use the PEM certificate
    ```
    https_port 3129 cert=/cygdrive/c/cacert.pem
    ```
