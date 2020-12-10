# How to set up a GREP11 server

## Hyper Protect Virtual Servers LPAR setup

## NOTE: All commands on this page are for reference only and not to be entered by you in the lab

Hyper Protect Virtual Servers runs in an LPAR that is defined in Secure Service Container (SSC) mode. Defining an LPAR is a normal task for an IBM Z or LinuxONE systems administrator, typically performed from the Hardware Management Console (HMC).

The systems administrator must[^1] dedicate one or more domains of one or more Crypto Express cards to the Hyper Protect Virtual Servers LPAR in order to use the GREP11 server. These Crypto Express cards must be defined in EP11 mode to your IBM Z or LinuxONE server in order to be used by a GREP11 server. 
These tasks are documented in the Hyper Protect Virtual Servers documentation, or in IBM publications referenced in the Hyper Protect Virtual Servers documentation. 

The version of Hyper Protect Virtual Servers that we will be using in this lab is version 1.2.2.1, which was released on November 6, 2020.

A GREP11 server communicates with one and only one Crypto Express domain, and vice versa.  You can run multiple GREP11 servers if you have multiple domains configured to your Hyper Protect Virtual Servers LPAR.

The GREP11 server is stateless, so desired levels of throughput and resilience can be achieved by loading the same master key in one or more domains of one or more Crypto Express 7S cards across one or more CECs across one or more geographies.

## Overview of GREP11 server setup

This section provides an overview of what the remaining sections of this page will discuss in detail.

The following steps are required:

1. Create a Certification Authority (CA) certificate and key

2. Create a GREP11 server X.509 certificate for [Transport Layer Security (TLS)](https://tools.ietf.org/html/rfc8446){target=_blank} authentication

3. Create an X.509 certificate for your client application for TLS authentication

4. List Crypto Domains on your Hyper Protect Virtual Servers LPAR

5. Create [YAML](https://yaml.org/){target=_blank} and/or [JSON](https://www.json.org/){target=_blank} configuration files for GREP11 server initialization

6. Start the GREP11 server

!!! Important
    **The commands shown on this page are for reference only**- they have already been performed by the lab instructors in order to set up the lab environment for you.

## Create a Certification Authority (CA) certificate and key

The *openssl* utility is used to generate a private key and a certificate that will act as a certification authority (CA). This will be used in later steps to issue certificates for the GREP11 server and for the client application which will connect to the GREP11 server.

Whenever you browse the web to a site with _https_, the server presents its certificate to your browser. This is known as _server-side_ or _one-way_ TLS authentication.  Most websites do not ask you to present a certificate. The website makes itself available to anybody who browses to it. 

With mutual TLS authentication, the client does need to present a certificate.  The server will only establish a session with a client who presents a certificate that is trusted by the server. This prevents our GREP11 server from being used by unauthorized clients.

The following command was used to create the RSA private key which will be used by our soon to be created CA

!!! Reminder
    **The command shown below is for reference only**- it has already been performed by the lab instructors in order to set up the lab environment for you.

``` bash
openssl genrsa -out atgz-hpvs-ca.key 2048
```

This created a file named `atgz-hpvs-ca.key` which is an RSA private key.  

In most cases (and mandatory from PKCS #11 version 2.4 onwards) the RSA private key contains enough information to reconstitute the public key. It is for this reason that the private key is used as input to the following command, which will create a Certification Authority X.509 certificate. X.509 certificates contain information identifying the certificate holder, the attributes of the certificate, including what the certificate can be used for, and, importantly, the public key.  

We used this command to create this certificate.

!!! Reminder
    **The command shown below is for reference only**- it has already been performed by the lab instructors in order to set up the lab environment for you.

``` bash
openssl req -new -x509 -key atgz-hpvs-ca.key -out atgz-hpvs-ca.pem
```

The `atgz-hpvs-ca.key` file was input to this command, and the `atgz-hpvs-ca.pem` file is the output of this command.  This `atgz-hpvs-ca.pem` file is our "homegrown" certification authority root certificate.  

I will use the Linux `cat` command to get a raw listing of the root certificate we just created:

!!! Reminder
    **The command shown below is for reference only**- it has already been performed by the lab instructors in order to set up the lab environment for you.

``` bash
cat atgz-hpvs-ca.pem
```

??? example "Example output"

    ```
    -----BEGIN CERTIFICATE-----
    MIIEMzCCAxugAwIBAgIUallVg6GrWPfoiwwUcCBdB+f7kPQwDQYJKoZIhvcNAQEL
    BQAwgagxCzAJBgNVBAYTAlVTMREwDwYDVQQIDAhWaXJnaW5pYTEQMA4GA1UEBwwH
    SGVybmRvbjEMMAoGA1UECgwDSUJNMSowKAYDVQQLDCFBZHZhbmNlZCBUZWNobm9s
    b2d5IEdyb3VwIC0gSUJNIFoxFjAUBgNVBAMMDUFURy1aIEhQVlMgQ0ExIjAgBgkq
    hkiG9w0BCQEWE3NpbGxpbWFuQHVzLmlibS5jb20wHhcNMjAxMjEwMTkyODI4WhcN
    MjIwMTA5MTkyODI4WjCBqDELMAkGA1UEBhMCVVMxETAPBgNVBAgMCFZpcmdpbmlh
    MRAwDgYDVQQHDAdIZXJuZG9uMQwwCgYDVQQKDANJQk0xKjAoBgNVBAsMIUFkdmFu
    Y2VkIFRlY2hub2xvZ3kgR3JvdXAgLSBJQk0gWjEWMBQGA1UEAwwNQVRHLVogSFBW
    UyBDQTEiMCAGCSqGSIb3DQEJARYTc2lsbGltYW5AdXMuaWJtLmNvbTCCASIwDQYJ
    KoZIhvcNAQEBBQADggEPADCCAQoCggEBAMAUfJAKBdbbCMn0N9DHvA4Chz/ihBBU
    xsCaTAUmJpVzDjv49rYTXY9YcxQpNYKyexsKjDlCfr+9gNlus1tKU0CrZD63ZLiA
    JCZZoomIZOoZQNquhj5f46/9x2d299FHxKY5z729hbHhHbjD+UkG60inyxS6Qm42
    OaILx8bjrrcHyOzhwKn4zm3RsaEOWcgqJP8yQvH8Bm7UElmKLADI5Ngp/+WWvNFE
    +QeXfYmKUSpLaJ0/4BD54Bbk780RSyOohvjv1O/JaLdHRXrJUYMSNNYQ3oK/7qRM
    CjOBnuaFY9FSdaGNvY2vS5vM44TaCy5DyMZ1GLCrCb9y13TvEVA9jFMCAwEAAaNT
    MFEwHQYDVR0OBBYEFN7BgpkfqiCTKIp/9JglwCDDgGJ5MB8GA1UdIwQYMBaAFN7B
    gpkfqiCTKIp/9JglwCDDgGJ5MA8GA1UdEwEB/wQFMAMBAf8wDQYJKoZIhvcNAQEL
    BQADggEBALFAM8/LQ9lkBAih7M6HrKR3efKh/Aj0UgRP6iYpprpW2H88Osw0lEnD
    ZZ2CTKeaHMDFHBUHapprLM3B0UnE/hchMxb4xQKihls6MRk9w3ZlZdUseStLYbS5
    LHkZWr2VnTjMg5n6QRaVEbJ/BE/65uzXAJ+3TukSRjeqnJUQlmOy+j3t6/hnuVhd
    5t9FI3bbF5Qi4ZpwpBnaAH74MMu/fvh7MXjUcwxgHOIvnY204FoNB5koGc4S0OK+
    sJ+dnL8L7J8Lq9mvi2+SmpU6lmWfAW09R5WsPomYIWKyTTplpt+4qJah87A4OSyJ
    mgvWNcD4Irl+wTtS0FixMdiLA5Z2+n0=
    -----END CERTIFICATE-----
    ```
The first and last lines are meant to assure you that this is a certificate, but the lines in between are less insightful, as it is [base64-encoded](https://en.wikipedia.org/wiki/Base64){target=_blank} binary data. Fortunately the *openssl* utility comes to our rescue and allows us to print the certificate in a form that a human can hope to understand:

!!! Reminder
    **The command shown below is for reference only**- it has already been performed by the lab instructors in order to set up the lab environment for you.

``` bash
openssl x509 -in atgz-hpvs-ca.pem -text
```

??? example "Example output"

    ``` bash  hl_lines="36-39 41-42"
    Certificate:
        Data:
            Version: 3 (0x2)
            Serial Number:
                6a:59:55:83:a1:ab:58:f7:e8:8b:0c:14:70:20:5d:07:e7:fb:90:f4
            Signature Algorithm: sha256WithRSAEncryption
            Issuer: C = US, ST = Virginia, L = Herndon, O = IBM, OU = Advanced Technology Group - IBM Z, CN = ATG-Z HPVS CA, emailAddress = silliman@us.ibm.com
            Validity
                Not Before: Dec 10 19:28:28 2020 GMT
                Not After : Jan  9 19:28:28 2022 GMT
            Subject: C = US, ST = Virginia, L = Herndon, O = IBM, OU = Advanced Technology Group - IBM Z, CN = ATG-Z HPVS CA, emailAddress = silliman@us.ibm.com
            Subject Public Key Info:
                Public Key Algorithm: rsaEncryption
                    RSA Public-Key: (2048 bit)
                    Modulus:
                        00:c0:14:7c:90:0a:05:d6:db:08:c9:f4:37:d0:c7:
                        bc:0e:02:87:3f:e2:84:10:54:c6:c0:9a:4c:05:26:
                        26:95:73:0e:3b:f8:f6:b6:13:5d:8f:58:73:14:29:
                        35:82:b2:7b:1b:0a:8c:39:42:7e:bf:bd:80:d9:6e:
                        b3:5b:4a:53:40:ab:64:3e:b7:64:b8:80:24:26:59:
                        a2:89:88:64:ea:19:40:da:ae:86:3e:5f:e3:af:fd:
                        c7:67:76:f7:d1:47:c4:a6:39:cf:bd:bd:85:b1:e1:
                        1d:b8:c3:f9:49:06:eb:48:a7:cb:14:ba:42:6e:36:
                        39:a2:0b:c7:c6:e3:ae:b7:07:c8:ec:e1:c0:a9:f8:
                        ce:6d:d1:b1:a1:0e:59:c8:2a:24:ff:32:42:f1:fc:
                        06:6e:d4:12:59:8a:2c:00:c8:e4:d8:29:ff:e5:96:
                        bc:d1:44:f9:07:97:7d:89:8a:51:2a:4b:68:9d:3f:
                        e0:10:f9:e0:16:e4:ef:cd:11:4b:23:a8:86:f8:ef:
                        d4:ef:c9:68:b7:47:45:7a:c9:51:83:12:34:d6:10:
                        de:82:bf:ee:a4:4c:0a:33:81:9e:e6:85:63:d1:52:
                        75:a1:8d:bd:8d:af:4b:9b:cc:e3:84:da:0b:2e:43:
                        c8:c6:75:18:b0:ab:09:bf:72:d7:74:ef:11:50:3d:
                        8c:53
                    Exponent: 65537 (0x10001)
            X509v3 extensions:
                X509v3 Subject Key Identifier: 
                    DE:C1:82:99:1F:AA:20:93:28:8A:7F:F4:98:25:C0:20:C3:80:62:79
                X509v3 Authority Key Identifier: 
                    keyid:DE:C1:82:99:1F:AA:20:93:28:8A:7F:F4:98:25:C0:20:C3:80:62:79

                X509v3 Basic Constraints: critical
                    CA:TRUE
        Signature Algorithm: sha256WithRSAEncryption
            b1:40:33:cf:cb:43:d9:64:04:08:a1:ec:ce:87:ac:a4:77:79:
            f2:a1:fc:08:f4:52:04:4f:ea:26:29:a6:ba:56:d8:7f:3c:3a:
            cc:34:94:49:c3:65:9d:82:4c:a7:9a:1c:c0:c5:1c:15:07:6a:
            9a:6b:2c:cd:c1:d1:49:c4:fe:17:21:33:16:f8:c5:02:a2:86:
            5b:3a:31:19:3d:c3:76:65:65:d5:2c:79:2b:4b:61:b4:b9:2c:
            79:19:5a:bd:95:9d:38:cc:83:99:fa:41:16:95:11:b2:7f:04:
            4f:fa:e6:ec:d7:00:9f:b7:4e:e9:12:46:37:aa:9c:95:10:96:
            63:b2:fa:3d:ed:eb:f8:67:b9:58:5d:e6:df:45:23:76:db:17:
            94:22:e1:9a:70:a4:19:da:00:7e:f8:30:cb:bf:7e:f8:7b:31:
            78:d4:73:0c:60:1c:e2:2f:9d:8d:b4:e0:5a:0d:07:99:28:19:
            ce:12:d0:e2:be:b0:9f:9d:9c:bf:0b:ec:9f:0b:ab:d9:af:8b:
            6f:92:9a:95:3a:96:65:9f:01:6d:3d:47:95:ac:3e:89:98:21:
            62:b2:4d:3a:65:a6:df:b8:a8:96:a1:f3:b0:38:39:2c:89:9a:
            0b:d6:35:c0:f8:22:b9:7e:c1:3b:52:d0:58:b1:31:d8:8b:03:
            96:76:fa:7d
    -----BEGIN CERTIFICATE-----
    MIIEMzCCAxugAwIBAgIUallVg6GrWPfoiwwUcCBdB+f7kPQwDQYJKoZIhvcNAQEL
    BQAwgagxCzAJBgNVBAYTAlVTMREwDwYDVQQIDAhWaXJnaW5pYTEQMA4GA1UEBwwH
    SGVybmRvbjEMMAoGA1UECgwDSUJNMSowKAYDVQQLDCFBZHZhbmNlZCBUZWNobm9s
    b2d5IEdyb3VwIC0gSUJNIFoxFjAUBgNVBAMMDUFURy1aIEhQVlMgQ0ExIjAgBgkq
    hkiG9w0BCQEWE3NpbGxpbWFuQHVzLmlibS5jb20wHhcNMjAxMjEwMTkyODI4WhcN
    MjIwMTA5MTkyODI4WjCBqDELMAkGA1UEBhMCVVMxETAPBgNVBAgMCFZpcmdpbmlh
    MRAwDgYDVQQHDAdIZXJuZG9uMQwwCgYDVQQKDANJQk0xKjAoBgNVBAsMIUFkdmFu
    Y2VkIFRlY2hub2xvZ3kgR3JvdXAgLSBJQk0gWjEWMBQGA1UEAwwNQVRHLVogSFBW
    UyBDQTEiMCAGCSqGSIb3DQEJARYTc2lsbGltYW5AdXMuaWJtLmNvbTCCASIwDQYJ
    KoZIhvcNAQEBBQADggEPADCCAQoCggEBAMAUfJAKBdbbCMn0N9DHvA4Chz/ihBBU
    xsCaTAUmJpVzDjv49rYTXY9YcxQpNYKyexsKjDlCfr+9gNlus1tKU0CrZD63ZLiA
    JCZZoomIZOoZQNquhj5f46/9x2d299FHxKY5z729hbHhHbjD+UkG60inyxS6Qm42
    OaILx8bjrrcHyOzhwKn4zm3RsaEOWcgqJP8yQvH8Bm7UElmKLADI5Ngp/+WWvNFE
    +QeXfYmKUSpLaJ0/4BD54Bbk780RSyOohvjv1O/JaLdHRXrJUYMSNNYQ3oK/7qRM
    CjOBnuaFY9FSdaGNvY2vS5vM44TaCy5DyMZ1GLCrCb9y13TvEVA9jFMCAwEAAaNT
    MFEwHQYDVR0OBBYEFN7BgpkfqiCTKIp/9JglwCDDgGJ5MB8GA1UdIwQYMBaAFN7B
    gpkfqiCTKIp/9JglwCDDgGJ5MA8GA1UdEwEB/wQFMAMBAf8wDQYJKoZIhvcNAQEL
    BQADggEBALFAM8/LQ9lkBAih7M6HrKR3efKh/Aj0UgRP6iYpprpW2H88Osw0lEnD
    ZZ2CTKeaHMDFHBUHapprLM3B0UnE/hchMxb4xQKihls6MRk9w3ZlZdUseStLYbS5
    LHkZWr2VnTjMg5n6QRaVEbJ/BE/65uzXAJ+3TukSRjeqnJUQlmOy+j3t6/hnuVhd
    5t9FI3bbF5Qi4ZpwpBnaAH74MMu/fvh7MXjUcwxgHOIvnY204FoNB5koGc4S0OK+
    sJ+dnL8L7J8Lq9mvi2+SmpU6lmWfAW09R5WsPomYIWKyTTplpt+4qJah87A4OSyJ
    mgvWNcD4Irl+wTtS0FixMdiLA5Z2+n0=
    -----END CERTIFICATE-----
    ```

A few lines of the output above have been highlighted. Notice the highlighted line that says `CA: TRUE`. This attribute indicates that this certificate is acting as a certification authority and can issue other certificates. Notice the lines above it which have values for *Subject Key Identifier* and *Authority Key Identifier*.  Subject Key Identifier is the identity of the holder of the certificate.  Authority Key Identifier is the identity of the issuer of the certificate. You can see that the values for these are the same.  This is known as a *self-signed certificate*.  Certification authorities (CA) exist in a hierarchy-  one CA can issue a certificate to another CA with the *CA: TRUE* attribute, and so forth.  In our lab we have this single, homegrown certification authority that we created, with its self-signed certificate that we created per the instructions in the Hyper Protect Virtual Servers documentation. 
   
## Create a GREP11 server X.509 certificate for TLS authentication

Once we created a "homegrown" certification authority, we next created an X.509 certificate for our GREP11 server. This certificate was issued by our certification authority.

The first step was to create another RSA private key that our GREP11 server will use:

!!! Reminder
    **The command shown below is for reference only**- it has already been performed by the lab instructors in order to set up the lab environment for you.

``` bash
openssl genrsa -out grep11-server80-9876-key.pem 2048
```

!!! note
    The value of the `-out` argument, `grep11-server80-9876-key.pem` can be whatever you want it to be. I named it what I did for a reason.  The _80_ in _server80_ is for the last octet of my Hyper Protect Virtual Servers LPAR's IP adresss, 192.168.22.80, and I intend to use this certificate for a GREP11 server listening on port 9876 on one of the LPAR's Crypto Express 7S domains.

Then, this private key was used as input to *openssl* in order to create a *certificate signing request*:

!!! Reminder
    **The command shown below is for reference only**- it has already been performed by the lab instructors in order to set up the lab environment for you.

``` bash
openssl req -new -key grep11-server80-9876-key.pem -out grep11-server80-9876.csr
```

The certificate signing request will be passed to our certification authority which will use the information to create an X.509 certificate.  We used *openssl* for this, too:

!!! Reminder
    **The command shown below is for reference only**- it has already been performed by the lab instructors in order to set up the lab environment for you.

```bash
openssl x509 -sha256 -req -in grep11-server80-9876.csr -CA atgz-hpvs-ca.pem -CAkey atgz-hpvs-ca.key -set_serial 8086 -extfile openssl.cnf -extensions server -days 365 -outform PEM -out grep11-server80-9876.pem
```

The file name of the certificate that was created by the preceding command is the value of the `-out` argument, `grep11-server80-9876.pem`.  `openssl` allows us to list this certificate in human-friendly form:

!!! Reminder
    **The command shown below is for reference only**- it has already been performed by the lab instructors in order to set up the lab environment for you.

``` bash
openssl x509 -in grep11-server80-9876.pem -noout -text
```

??? example "Example output"
 
    ```
    Certificate:
        Data:
            Version: 3 (0x2)
            Serial Number: 8086 (0x1f96)
            Signature Algorithm: sha256WithRSAEncryption
            Issuer: C = US, ST = Virginia, L = Herndon, O = IBM, OU = Advanced Technology Group - IBM Z, CN = ATG-Z HPVS CA, emailAddress = silliman@us.ibm.com
            Validity
                Not Before: Dec 10 19:28:28 2020 GMT
                Not After : Jan  4 19:28:28 2022 GMT
            Subject: C = US, ST = Virginia, L = Herndon, O = IBM, OU = Advanced Technology Group - IBM Z, CN = 192.168.22.80
            Subject Public Key Info:
                Public Key Algorithm: rsaEncryption
                    RSA Public-Key: (2048 bit)
                    Modulus:
                        00:d9:c4:97:86:cc:d9:a8:d7:0e:e4:d5:7e:6a:97:
                        e6:03:f4:85:4c:4d:2f:c5:bc:1e:88:da:ab:31:68:
                        3b:1b:b7:eb:ae:81:a6:0f:34:1f:b0:90:bb:a7:1a:
                        7a:2c:c1:6b:be:b6:8b:9d:b4:b8:71:78:ff:fb:84:
                        61:c4:22:4e:8f:42:52:90:15:de:58:c8:1f:3d:5d:
                        3c:15:c7:9f:9d:bb:13:c4:0c:d8:a1:53:83:56:18:
                        ec:75:fd:69:5a:f4:ed:6e:61:5e:f0:1b:57:58:7c:
                        89:28:5b:b7:81:c9:6d:4e:fa:63:33:a1:98:ee:7a:
                        18:5b:ad:c5:53:8a:fe:3e:c7:99:07:7f:45:e2:15:
                        fa:2a:bc:e2:eb:2a:dc:7e:38:7a:8c:ec:1f:89:b1:
                        e3:91:6f:1a:36:d3:17:03:54:c3:56:57:65:7f:d4:
                        6b:56:dc:94:21:d9:5d:43:06:26:a6:fa:48:06:85:
                        95:e9:f3:10:b3:26:cb:69:c3:67:28:d3:ef:74:40:
                        50:7b:2a:f9:56:20:79:e4:92:64:2d:ea:6b:bc:07:
                        2a:95:3e:d2:80:5e:c5:61:b7:84:ca:63:83:e0:0a:
                        67:fe:e3:9c:af:12:54:f0:22:82:b3:84:30:87:08:
                        b5:8c:bf:05:af:dd:80:94:a3:0f:39:d4:d7:2e:d5:
                        d3:e7
                    Exponent: 65537 (0x10001)
            X509v3 extensions:
                X509v3 Basic Constraints: 
                    CA:FALSE
                X509v3 Key Usage: 
                    Digital Signature, Key Encipherment, Data Encipherment
                X509v3 Extended Key Usage: 
                    TLS Web Server Authentication
                Netscape Cert Type: 
                    SSL Server
                X509v3 CRL Distribution Points: 

                    Full Name:
                    URI:http://localhost/ca.crl

                X509v3 Subject Alternative Name: 
                    DNS:192.168.22.80:9876, IP Address:192.168.22.80
        Signature Algorithm: sha256WithRSAEncryption
            1f:bb:b0:26:34:62:82:62:ca:7f:1c:a6:ef:54:15:d9:44:88:
            e4:97:19:5b:2c:fc:dd:1c:01:70:ee:27:1c:ec:49:58:25:a5:
            dc:9e:9a:55:71:9b:05:bb:c1:1b:6e:85:a3:ab:9c:ce:05:bf:
            29:3b:cb:ed:98:f5:01:e8:cb:e2:e6:14:c1:74:0d:96:38:14:
            58:ca:84:ad:af:35:fd:12:50:16:10:f3:6d:c1:fe:ba:04:12:
            ea:19:17:dd:95:a8:8b:65:83:da:bc:ef:6e:23:00:2c:52:6e:
            80:af:a9:93:7a:5f:40:8d:e7:55:ea:1d:90:d8:8c:04:59:91:
            91:b4:22:19:9f:9a:b6:20:ee:36:4d:c1:75:29:54:e1:a0:47:
            cd:82:14:86:75:37:60:4e:1a:54:f5:81:ec:5a:3a:de:64:93:
            51:fb:c6:44:b1:66:d7:41:59:b6:95:27:c9:81:fc:d9:76:a5:
            f2:16:74:47:8c:c8:a3:b1:1d:f3:98:34:b1:fa:52:d9:3e:d1:
            63:8b:70:d3:a3:aa:ac:47:1c:90:13:76:df:98:47:ae:19:e2:
            c4:d5:81:eb:44:66:45:09:6a:75:c6:5f:0c:aa:e0:44:52:16:
            35:be:49:f0:10:65:3f:df:a3:50:0d:3b:ae:94:59:9c:28:a5:
            41:24:a5:45
    ```

## Create an X.509 certificate for your client application for TLS authentication

The client applications need a certificate that is issued by the same certification authority that the GREP11 server uses. The process is similar- create a private key, pass that as input for the creation of a certificte signing request, and then pass that to the certification authority in order to receive a certificate.  We used these commands:

!!! Reminder
    **The command shown below is for reference only**- it has already been performed by the lab instructors in order to set up the lab environment for you.

``` bash
openssl genrsa -out client-key.pem 2048
```

!!! Reminder
    **The command shown below is for reference only**- it has already been performed by the lab instructors in order to set up the lab environment for you.

``` bash
openssl req -new -key client-key.pem -out client.csr
```

!!! Reminder
    **The command shown below is for reference only**- it has already been performed by the lab instructors in order to set up the lab environment for you.

``` bash
openssl x509 -req -days 1000 -in client.csr -CA ca.pem -CAcreateserial -CAkey ca.key -out client.pem
```

Here is a listing of the client certificate that you will be using in the lab. It will be configured properly for you so that your client program can present it to the GREP11 server:

!!! Reminder
    **The command shown below is for reference only**- it has already been performed by the lab instructors in order to set up the lab environment for you.

``` bash
openssl x509 -in client.pem -noout -text
```

??? example "Example Output"

    ``` hl_lines="11"
    Certificate:
        Data:
            Version: 1 (0x0)
            Serial Number:
                7a:dc:83:ab:72:46:2f:30:b9:d2:71:b4:9b:00:43:4f:53:63:a7:ce
            Signature Algorithm: sha256WithRSAEncryption
            Issuer: C = US, ST = Virginia, L = Herndon, O = IBM, OU = Advanced Technology Group - IBM Z, CN = ATG-Z HPVS CA, emailAddress = silliman@us.ibm.com
            Validity
                Not Before: Dec 10 19:28:28 2020 GMT
                Not After : Mar 20 19:28:28 2021 GMT
            Subject: C = US, ST = Virginia, L = Herndon, O = IBM, OU = Advanced Technology Group - IBM Z, CN = GREP11 Lab Student, emailAddress = silliman@us.ibm.com
            Subject Public Key Info:
                Public Key Algorithm: rsaEncryption
                    RSA Public-Key: (2048 bit)
                    Modulus:
                        00:d2:4f:86:c1:d6:9a:6b:f4:2c:7d:6f:91:59:b9:
                        98:12:6e:d1:5f:ea:ca:90:b4:74:0e:dd:99:ce:20:
                        ca:f6:75:df:fe:54:76:ef:fd:3a:a2:f9:cf:15:0d:
                        1e:19:5c:e5:0f:55:50:ff:64:7f:2b:a6:1f:35:8b:
                        38:f8:f4:2a:a6:c7:66:0c:ef:17:d9:40:46:fb:d1:
                        ca:28:dd:a0:6b:08:28:b9:55:95:9d:cc:67:9b:f4:
                        26:da:04:76:fb:d3:02:e3:8e:af:f7:c3:4c:01:30:
                        04:47:e3:e3:0a:58:33:47:8e:11:50:1d:1a:ca:30:
                        a8:09:76:fc:b9:a7:86:05:2b:46:44:f6:dd:48:e2:
                        5e:64:78:8b:06:42:32:5f:cf:7d:e6:80:e1:32:94:
                        6f:fe:8a:b9:f5:a1:3a:c0:d5:3a:c7:5f:4e:e3:7b:
                        2e:0f:58:a2:35:26:57:0c:0d:c7:7f:2d:8f:8c:09:
                        84:ee:eb:d4:0e:da:5a:3a:e3:09:2f:f2:be:b9:b0:
                        48:cd:c7:d0:4c:03:d6:25:68:5d:5f:7e:3a:b1:9b:
                        4b:a4:09:b3:40:30:be:6d:be:9c:de:77:e7:18:4c:
                        ff:6d:cf:70:9c:37:bd:8e:5b:6d:75:1f:a5:c2:55:
                        ec:38:25:c9:cd:38:4b:8e:63:47:63:4e:e7:90:21:
                        6b:a1
                    Exponent: 65537 (0x10001)
        Signature Algorithm: sha256WithRSAEncryption
            ac:89:54:fc:40:06:1f:84:99:08:f8:70:ab:b0:24:1d:b3:6c:
            a2:86:bd:b8:82:e3:6d:aa:c1:c7:7a:b8:3c:1d:59:80:89:92:
            57:07:ca:28:cd:80:de:11:e7:ed:d9:40:29:0b:90:1a:b7:9b:
            ed:b5:15:6a:6c:1c:31:3f:82:0f:01:1f:64:05:bb:12:16:67:
            d2:b2:3d:84:ef:74:34:56:80:11:69:ab:85:5b:43:ac:ba:13:
            e2:ca:b3:12:4a:39:ff:f1:4f:47:60:e9:16:49:75:34:95:1d:
            e0:51:c4:05:96:5e:1b:50:31:cf:ba:1a:a5:e1:1f:ed:8b:62:
            84:6d:a4:f3:6c:d4:20:d1:f1:b1:6e:79:de:57:c9:93:f7:12:
            68:86:45:dd:f4:2d:a9:c9:41:b4:6c:5e:79:01:5a:91:64:a1:
            01:14:c5:81:04:fb:64:7e:d1:42:ef:2c:e8:9f:6c:ba:b9:01:
            53:4b:32:30:0f:2a:30:4d:84:d1:11:a6:b6:cc:56:58:b0:1c:
            4f:ea:dc:3e:6e:4f:60:5b:ed:a1:0d:c8:f1:6b:dc:1a:05:00:
            ad:a9:72:ce:ff:d1:d6:ae:0a:a9:56:36:5a:ab:d0:33:41:59:
            80:8f:17:90:8a:71:f6:df:bd:6c:8e:4b:10:5d:5b:8f:94:9a:
            2d:03:85:0b
    ```

!!! note
    For the lab, each student will use a copy of the same client certificate. In most production use cases each client would use their own certificate which uniquely identifies them.

## List Crypto Domains on your Hyper Protect Virtual Servers LPAR

You can list your available domains with the Hyper Protect Virtual Server CLI:

!!! Reminder
    **The command shown below is for reference only**- it has already been performed by the lab instructors in order to set up the lab environment for you.

``` bash
hpvs crypto list
```

???+ example "Example output"

    ```
     hpvs crypto list
    +---------------+--------+
    | CRYPTO.DOMAIN | STATUS |
    +---------------+--------+
    | 08.0016       | online |
    | 0a.0016       | online |
    +---------------+--------+
    ```

The values shown in the **CRYPTO.DOMAIN** column are in hexadecimal.  So, translating to decimal, our LPAR is using Crypto Express 7S cards 8 and 10, with domain 22 in both cards assigned to our LPAR.  (A Crypto Express 7S card can have up to 85 domains- think of each domain as a "virtual" Crypto Express 7S card).

## Create YAML or JSON configuration files for GREP11 server

There are two alternative ways to start your GREP11 server:

1. Using `hpvs deploy` with a YAML file as input
2. Using `hpvs vs create` with a JSON file as input

I will show both the YAML and JSON files first, then I will show both methods of starting the GREP11 server, and then I'll explain the difference between the two start methods.

### YAML file for GREP11 server configuration

This is the YAML file for a GREP11 server that will listen for client connections on port 9876, and will use domain 22 (hex 16) of Crypto Express 7S card 08:

??? example "GREP11 server YAML configuration file"

    ``` bash hl_lines="16 19 38"
    version: v1 
    #
    # use this file with the 'hpvs deploy' command, e.g.,
    #
    #  hpvs deploy --config $HOME/hpvs/config/grep11/vs_grep11_80-9876.yml 
    #
    type: virtualserver
    virtualservers:
    - name: grep11-08-0016-9876
    host: atgzlpar80
    repoid: hpcsKpGrep11_runq
    imagetag: 1.2.1
    imagefile: hpcsKpGrep11_runq.tar.gz
    crypto:
        crypto_matrix:
        - 08.0016
    environment:
    - key: EP11SERVER_EP11CRYPTO_DOMAIN
        value: "08.0016"
    #
    - key: EP11SERVER_EP11CRYPTO_CONNECTION_TLS_CERTFILEBYTES
        value: "@/home/hyper-protect-lab/hpvs/config/grep11/keys/grep11-server80-9876.pem"
    - key: EP11SERVER_EP11CRYPTO_CONNECTION_TLS_KEYFILEBYTES
        value: "@/home/hyper-protect-lab/hpvs/config/grep11/keys/grep11-server80-9876-key.pem"
    - key: EP11SERVER_EP11CRYPTO_CONNECTION_TLS_CACERTBYTES
        value: "@/home/hyper-protect-lab/hpvs/config/grep11/keys/atgz-hpvs-ca.pem"
    - key: EP11SERVER_EP11CRYPTO_CONNECTION_TLS_ENABLED
        value: "true"
    - key: EP11SERVER_EP11CRYPTO_CONNECTION_TLS_MUTUAL
        value: "true"
    - key: EP11SERVER_EP11CRYPTO_ENABLED
        value: "true"
    - key: TLS_GRPC_CERTS_DOMAIN_CRT
        value: "\\n"
    - key: TLS_GRPC_CERTS_DOMAIN_KEY
        value: "\\n"
    - key: TLS_GRPC_CERTS_ROOTCA_CRT
        value: "\\n"
    ports:
    - hostport: 9876
        protocol: tcp
        containerport: 9876
    ```

### JSON file for GREP11 server configuration

This is a JSON file for a GREP11 server that will listen for client connections on port 9876, and will use domain 22 (hex 16) of Crypto Express 7S card 08:

??? example "GREP11 server JSON configuration file"

    ``` hl_lines="2"
    {
    "EP11SERVER_EP11CRYPTO_DOMAIN":"08.0016",
    "EP11SERVER_EP11CRYPTO_CONNECTION_TLS_CERTFILEBYTES":"@/home/hyper-protect-lab/hpvs/config/grep11/keys/grep11-server80-9876.pem",
    "EP11SERVER_EP11CRYPTO_CONNECTION_TLS_KEYFILEBYTES":"@/home/hyper-protect-lab/hpvs/config/grep11/keys/grep11-server80-9876-key.pem",
    "EP11SERVER_EP11CRYPTO_CONNECTION_TLS_CACERTBYTES":"@/home/hyper-protect-lab/hpvs/config/grep11/keys/atgz-hpvs-ca.pem",
    "EP11SERVER_EP11CRYPTO_CONNECTION_TLS_ENABLED":true,
    "EP11SERVER_EP11CRYPTO_CONNECTION_TLS_MUTUAL":true,
    "EP11SERVER_EP11CRYPTO_ENABLED":true,
    "TLS_GRPC_CERTS_DOMAIN_CRT":"\\n",
    "TLS_GRPC_CERTS_DOMAIN_KEY":"\\n",
    "TLS_GRPC_CERTS_ROOTCA_CRT":"\\n"
    }
    ```

## Start the GREP11 server

If you looked carefully at the JSON file and the YAML file in the previous section, you may have noticed that the YAML file contains more information than the JSON file contains.  As a result, the syntax is simpler for the command that uses the YAML file.  The command that uses the JSON file uses command line arguments to provide some of the information that the YAML file contained.  Compare the two methods:

!!! Reminder
    **The commands shown below are for reference only**- they have already been performed by the lab instructors in order to set up the lab environment for you.

???+ example "Command to start the GREP11 server with the YAML file"

    ```
    hpvs deploy --config $HOME/hpvs/config/grep11/vs_grep11_80-9876.yml
    ```

???+ example "Command to start the GREP11 server with the JSON file"

    ```
    hpvs vs create --name grep11-08-0016-9876 --repo hpcsKpGrep11_runq --tag 1.2.2.1 --crypto_matrix=08.0016 --cpu 2 --ram 2048 --envjsonpath ${HOME}/hpvs/config/grep11/grep11_env_08.0016.json --ports "{containerport = 9876, protocol = tcp, hostport = 9876}"
    ```

!!! note "Difference between the two commands"
    I like the simplicity of the `hpvs deploy`method much better than the long syntax of the `hpvs vs create` command.  But the `hpvs vs create` command has a benefit-  the `hpvs deploy` command uploads the GREP11 server Docker image from your workstation where you run the CLI to the Hyper Protect Virtual Servers LPAR, every single time.  This Docker image only needs to be sent up once.  The `hpvs vs create` command is smart enough to have this figured out and not do the unnecessary upload of the Docker image the second and subsequent times you run it.

!!! Important
    **Starting now, as you navigate to the next section of the lab, you should enter all the commands shown in the lab.**  Only the commands in this section were for reference.
    
[^1]: The GREP11 server is a feature provided by Hyper Protect Virtual Servers. You are not required to use it. If you do not use this feature, you do not have to define Crypto Express domains to the LPAR.  You may wish to for other purposes, and you can use the other modes offered by Crypto Express cards for those purposes, but you must use EP11 mode for usage by the GREP11 server.
