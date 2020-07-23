# How to set up a GREP11 server

## Hyper Protect Virtual Servers LPAR setup

Hyper Protect Virtual Servers runs in an LPAR that is defined in Secure Service Container (SSC) mode. Defining an LPAR is a normal task for an IBM Z or LinuxONE systems administrator, typically performed from the Hardware Management Console (HMC).

The systems administrator must[^1] dedicate one or more domains of one or more Crypto Express cards to the Hyper Protect Virtual Servers LPAR in order to use the GREP11 server. These Crypto Express cards must be defined in EP11 mode to your IBM Z or LinuxONE server in order to be used by a GREP11 server. 
These tasks are documented in the Hyper Protect Virtual Servers documentation, or in IBM publications referenced in the Hyper Protect Virtual Servers documentation. 

The version of Hyper Protect Virtual Servers that we will be using in this lab is version 1.2.1, which became generally available on July 17, 2020.

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
    The commands shown on this page are for reference only- they have already been performed by the lab instructors in order to set up the lab environment for you.  

## Create a Certification Authority (CA) certificate and key

The *openssl* utility is used to generate a private key and a certificate that will act as a certification authority (CA). This will be used in later steps to issue certificates for the GREP11 server and for the client application which will connect to the GREP11 server.

Whenever you browse the web to a site with _https_, the server presents its certificate to your browser. This is known as _server-side_ or _one-way_ TLS authentication.  Most websites do not ask you to present a certificate. The website makes itself available to anybody who browses to it. 

With mutual TLS authentication, the client does need to present a certificate.  The server will only establish a session with a client who presents a certificate that is trusted by the server. This prevents our GREP11 server from being used by unauthorized clients.

The following command was used to create the RSA private key which will be used by our soon to be created CA

``` bash
openssl genrsa -out ca.key 2048
```

This created a file named `ca.key` which is an RSA private key.  

In most cases (and mandatory from PKCS #11 version 2.4 onwards) the RSA private key contains enough information to reconstitute the public key. It is for this reason that the private key is used as input to the following command, which will create a Certification Authority X.509 certificate. X.509 certificates contain information identifying the certificate holder, the attributes of the certificate, including what the certificate can be used for, and, importantly, the public key.  

We used this command to create this certificate.

``` bash
openssl req -new -x509 -key ca.key -out ca.pem
```

The `ca.key` file was input to this command, and the `ca.pem` file is the output of this command.  This `ca.pem` file is our "homegrown" certification authority root certificate.  

I will use the Linux `cat` command to get a raw listing of the root certificate we just created:

``` bash
cat cat.pem
```

??? example "Example output"

    ```
    -----BEGIN CERTIFICATE-----
    MIIDcDCCAlgCCQDrH6CQd+lq9DANBgkqhkiG9w0BAQsFADB6MQswCQYDVQQGEwJV
    UzERMA8GA1UECAwIVmlyZ2luaWExEDAOBgNVBAcMB0hlcm5kb24xDDAKBgNVBAoM
    A0lCTTEmMCQGA1UECwwdSUJNIFdhc2hpbmd0b24gU3lzdGVtcyBDZW50ZXIxEDAO
    BgNVBAMMB3dzY2hwdnMwHhcNMjAwNzEyMTQ0MDUzWhcNMjEwODExMTQ0MDUzWjB6
    MQswCQYDVQQGEwJVUzERMA8GA1UECAwIVmlyZ2luaWExEDAOBgNVBAcMB0hlcm5k
    b24xDDAKBgNVBAoMA0lCTTEmMCQGA1UECwwdSUJNIFdhc2hpbmd0b24gU3lzdGVt
    cyBDZW50ZXIxEDAOBgNVBAMMB3dzY2hwdnMwggEiMA0GCSqGSIb3DQEBAQUAA4IB
    DwAwggEKAoIBAQDRxnTkDFqdSmgDGlsrKHljg9+Lbf8HLLwuopXzdLukCoGvlIAf
    SzZPBsHd4JbMnYLqsxQzLDC1CGb7mz0wzk75wzy4yyhKXFqsEsSoZNMtA0HzWghZ
    MVrebaBh27oiKHNBMwZWTHeW4GkiVQFjDwXr63VKv6R03I9CdxARyErfF6qLp++e
    CoQz43BLVbaT9wCzRcqChg5BhnIwBuO9cghJLiWT9EalQ0mndr8YSwkWfhcOniqA
    iB6mHRKRLh0SoEL1Zoqbf585SpysjkRRM8yjB4Ju4EnyiLxmTDjjuaJ8CUOuNz+m
    PhJf4bax9sOO4IVI63jHpmSJUVR2yiUB0HYVAgMBAAEwDQYJKoZIhvcNAQELBQAD
    ggEBAH2amw0xySsZj4NNo5QYVN209wUqRGSjsNCyIqO6j0k2Sastm1nxX8Yv7hNJ
    WUqgKpHSLQVCOpgld6V7YZnBd+53Iq87iymbQ1oA5D5bjoQLzAtRHHku6Kf0p08D
    NCXQOS197Z41hNmMYT1sqbgSv+z2aahCoQKD3Wdh0lXHL621UEMk3Qo5SqfZ+RCm
    m69v5KpG7YT3irABSAkYi8z2dN/lzHVfgEUa7+Im4NEVVCQwBld/+hX/64pd2Op7
    tyvWa0MXc0O//D5NLNVyzZJcnps2ru1P6IeBEEsdfwkVNEBqUXxK4i5eFBkar5Qx
    OCqkCywplgKoAyh8SGTPIkS4xhw=
    -----END CERTIFICATE-----
    ```
The first and last lines are meant to assure you that this is a certificate, but the lines in between are less insightful, as it is [base64-encoded](https://en.wikipedia.org/wiki/Base64){target=_blank} binary data. Fortunately the *openssl* utility comes to our rescue and allows us to print the certificate in a form that both geek- and human-readable form:

``` bash
openssl x509 -in ca.pem -text
```

??? example "Example output"

    ``` bash  hl_lines="36-39 41-42"
    Certificate:
        Data:
            Version: 3 (0x2)
            Serial Number:
                42:85:e5:33:4e:1a:e2:d4:9e:ed:8f:1d:27:2f:f1:10:29:ed:31:67
            Signature Algorithm: sha256WithRSAEncryption
            Issuer: C = US, ST = Virginia, L = Herndon, O = IBM, OU = IBM Washington Systems Center, CN = wschpvs
            Validity
                Not Before: Jul 23 21:36:44 2020 GMT
                Not After : Aug 22 21:36:44 2021 GMT
            Subject: C = US, ST = Virginia, L = Herndon, O = IBM, OU = IBM Washington Systems Center, CN = wschpvs
            Subject Public Key Info:
                Public Key Algorithm: rsaEncryption
                    RSA Public-Key: (2048 bit)
                    Modulus:
                        00:dd:b6:fa:09:ab:21:6a:6c:fa:0b:e0:49:f9:f5:
                        b6:fa:f8:4f:26:10:cf:ef:11:2a:16:c3:3a:cb:3f:
                        9f:da:d1:62:2b:2e:e4:5f:c3:b6:ab:29:29:54:b7:
                        9e:97:4e:59:e3:d1:b2:51:27:f9:02:2f:dd:33:95:
                        a9:9d:14:63:c1:ea:6e:04:f4:5a:d2:51:d9:e7:ca:
                        31:01:5a:86:09:94:9b:0d:1e:03:3b:cb:ae:51:df:
                        18:ec:ec:86:a2:8f:34:6f:bd:80:c5:a6:af:be:c7:
                        0b:d1:1b:dd:e0:9c:7d:a3:4b:00:65:a3:a5:fe:1c:
                        bc:8b:fa:02:9d:7a:0d:7e:52:d9:6d:44:1d:5f:d7:
                        2f:93:6b:f6:b8:e6:66:05:43:55:8d:9c:d6:ec:96:
                        18:0b:71:c8:c4:ad:fe:08:18:4e:e6:33:e9:5d:c9:
                        c7:83:0b:83:2f:c4:85:7d:40:1f:6f:76:41:9f:c9:
                        50:15:7a:cc:57:94:df:ec:b5:5e:08:1f:95:09:ab:
                        d6:b3:68:bb:df:39:6f:f1:ba:f3:e2:90:97:27:55:
                        15:5a:03:82:48:f6:ca:02:f9:90:55:f4:11:cd:31:
                        34:68:87:22:8b:6e:39:fc:53:ad:2e:26:9d:24:27:
                        31:37:25:bd:b2:4b:fb:a7:57:9c:86:d6:d8:bf:4a:
                        ad:eb
                    Exponent: 65537 (0x10001)
            X509v3 extensions:
                X509v3 Subject Key Identifier: 
                    9B:D8:7D:1B:C2:7F:44:21:CB:18:54:0D:0A:E2:B6:E4:78:F4:F3:E5
                X509v3 Authority Key Identifier: 
                    keyid:9B:D8:7D:1B:C2:7F:44:21:CB:18:54:0D:0A:E2:B6:E4:78:F4:F3:E5

                X509v3 Basic Constraints: critical
                    CA:TRUE
        Signature Algorithm: sha256WithRSAEncryption
            6e:ca:cf:a8:f8:0c:bf:59:d8:c9:85:48:bc:d8:3c:7a:a0:da:
            18:b7:23:b2:40:ce:df:44:a9:f1:e2:8d:0e:c1:e3:b7:00:48:
            71:81:01:b9:42:b8:61:87:9b:85:4e:ca:27:71:b5:d1:c4:46:
            cd:57:91:2a:04:54:b9:ab:f5:df:82:16:aa:80:11:27:7b:ba:
            ef:11:d3:88:b1:46:d3:43:32:fb:ac:04:3c:9d:f1:c6:53:74:
            36:bd:e2:cb:ff:d2:a6:56:fa:3c:97:f6:e9:7c:63:93:2a:46:
            d4:1c:bf:c1:86:4e:52:9d:f6:0a:b4:73:21:7b:52:93:0f:3b:
            d5:83:f1:f8:22:ec:6a:02:26:22:6f:d7:e0:0d:5f:83:94:40:
            46:f6:5e:af:bc:c1:9b:86:2c:aa:e8:95:59:f6:11:31:0a:5a:
            1a:36:69:b8:b0:11:32:61:cb:16:3b:3d:7f:56:08:1a:5f:5b:
            15:e2:4e:2f:c8:46:4a:ce:02:12:90:af:74:b9:81:34:0c:dd:
            b7:0f:fc:88:49:d6:37:d0:25:72:4d:e1:b5:69:a0:79:b4:38:
            02:47:c1:30:cf:6b:14:12:e8:d8:b9:e2:c6:68:60:00:6a:e8:
            14:08:bd:7f:5c:df:d9:73:1a:51:f8:b9:4a:a6:0a:b8:6c:04:
            bf:34:94:49
    -----BEGIN CERTIFICATE-----
    MIID1TCCAr2gAwIBAgIUQoXlM04a4tSe7Y8dJy/xECntMWcwDQYJKoZIhvcNAQEL
    BQAwejELMAkGA1UEBhMCVVMxETAPBgNVBAgMCFZpcmdpbmlhMRAwDgYDVQQHDAdI
    ZXJuZG9uMQwwCgYDVQQKDANJQk0xJjAkBgNVBAsMHUlCTSBXYXNoaW5ndG9uIFN5
    c3RlbXMgQ2VudGVyMRAwDgYDVQQDDAd3c2NocHZzMB4XDTIwMDcyMzIxMzY0NFoX
    DTIxMDgyMjIxMzY0NFowejELMAkGA1UEBhMCVVMxETAPBgNVBAgMCFZpcmdpbmlh
    MRAwDgYDVQQHDAdIZXJuZG9uMQwwCgYDVQQKDANJQk0xJjAkBgNVBAsMHUlCTSBX
    YXNoaW5ndG9uIFN5c3RlbXMgQ2VudGVyMRAwDgYDVQQDDAd3c2NocHZzMIIBIjAN
    BgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA3bb6Cashamz6C+BJ+fW2+vhPJhDP
    7xEqFsM6yz+f2tFiKy7kX8O2qykpVLeel05Z49GyUSf5Ai/dM5WpnRRjwepuBPRa
    0lHZ58oxAVqGCZSbDR4DO8uuUd8Y7OyGoo80b72AxaavvscL0Rvd4Jx9o0sAZaOl
    /hy8i/oCnXoNflLZbUQdX9cvk2v2uOZmBUNVjZzW7JYYC3HIxK3+CBhO5jPpXcnH
    gwuDL8SFfUAfb3ZBn8lQFXrMV5Tf7LVeCB+VCavWs2i73zlv8brz4pCXJ1UVWgOC
    SPbKAvmQVfQRzTE0aIcii245/FOtLiadJCcxNyW9skv7p1echtbYv0qt6wIDAQAB
    o1MwUTAdBgNVHQ4EFgQUm9h9G8J/RCHLGFQNCuK25Hj08+UwHwYDVR0jBBgwFoAU
    m9h9G8J/RCHLGFQNCuK25Hj08+UwDwYDVR0TAQH/BAUwAwEB/zANBgkqhkiG9w0B
    AQsFAAOCAQEAbsrPqPgMv1nYyYVIvNg8eqDaGLcjskDO30Sp8eKNDsHjtwBIcYEB
    uUK4YYebhU7KJ3G10cRGzVeRKgRUuav134IWqoARJ3u67xHTiLFG00My+6wEPJ3x
    xlN0Nr3iy//Splb6PJf26XxjkypG1By/wYZOUp32CrRzIXtSkw871YPx+CLsagIm
    Im/X4A1fg5RARvZer7zBm4YsquiVWfYRMQpaGjZpuLARMmHLFjs9f1YIGl9bFeJO
    L8hGSs4CEpCvdLmBNAzdtw/8iEnWN9Alck3htWmgebQ4AkfBMM9rFBLo2Lnixmhg
    AGroFAi9f1zf2XMaUfi5SqYKuGwEvzSUSQ==
    -----END CERTIFICATE-----
    ```

A few lines of the output above have been highlighted. Notice the highlighted line that says `CA: TRUE`. This attribute indicates that this certificate is acting as a certification authority and can issue other certificates. Notice the lines above it which have values for *Subject Key Identifier* and *Authority Key Identifier*.  Subject Key Identifier is the identity of the holder of the certificate.  Authority Key Identifier is the identity of the issuer of the certificate. You can see that the values for these are the same.  This is known as a *self-signed certificate*.  Certification authorities (CA) exist in a hierarchy-  one CA can issue a certificate to another CA with the *CA: TRUE* attribute, and so forth.  In our lab we have this single, homegrown certification authority that we created, with its self-signed certificate that we created per the instructions in the Hyper Protect Virtual Servers documentation. 
   
## Create a GREP11 server X.509 certificate for TLS authentication

Once we created a "homegrown" certification authority, we next created an X.509 certificate for our GREP11 server. This certificate was issued by our certification authority.

The first step was to create another RSA private key that our GREP11 server will use:

``` bash
openssl genrsa -out server80-9876-19876-key.pem 2048
```

!!! note
    The value of the `-out` argument, `server80-9876-19876-key.pem` can be whatever you want it to be. I named it what I did for a reason.  the _80_ in _server80_ is for the last octet of my Hyper Protect Virtual Servers LPAR's IP adresss, 192.168.22.80, and I intend to use this certificate for two GREP11 servers, one listening on port 9876 on one of the LPAR's Crypto Express 7S domains, and the second server will listen on port 19876 on the LPAR's second Crypto Express 7S domain.

Then, this private key was used as input to *openssl* in order to create a *certificate signing request*:

``` bash
openssl req -new -key server80-9876-19876-key.pem -out server80-9876-19876.csr
```

The certificate signing request will be passed to our certification authority which will use the information to create an X.509 certificate.  We used *openssl* for this, too:

```bash
openssl x509 -sha256 -req -in server80-9876-19876.csr -CA ca.pem -CAkey ca.key -set_serial 8086 -extfile openssl.cnf -extensions server -days 365 -outform PEM -out server80-9876-19876.pem
```

The file name of the certificate that was created by the preceding command is the value of the `-out` argument, `server80-9876-19876.pem`.  `openssl` allows us to list this certificate in human-friendly form:

``` bash
openssl x509 -in server80-9876-19876.pem -noout -text
```

??? example "Example output"
 
    ```
    Certificate:
        Data:
            Version: 3 (0x2)
            Serial Number: 8086 (0x1f96)
            Signature Algorithm: sha256WithRSAEncryption
            Issuer: C = US, ST = Virginia, L = Herndon, O = IBM, OU = IBM Washington Systems Center, CN = wschpvs
            Validity
                Not Before: Jul 23 21:47:22 2020 GMT
                Not After : Jul 23 21:47:22 2021 GMT
            Subject: C = US, ST = Virginia, L = Herndon, O = IBM, OU = IBM Washington Systems Center, CN = 192.168.22.80
            Subject Public Key Info:
                Public Key Algorithm: rsaEncryption
                    RSA Public-Key: (2048 bit)
                    Modulus:
                        00:b8:1a:5d:37:d6:c5:5b:2a:40:90:e5:e9:7f:c7:
                        19:e4:83:fd:ce:f7:1b:f7:37:09:c1:f6:1f:03:10:
                        78:70:52:27:66:1c:49:20:75:03:12:2f:7c:1b:ea:
                        7b:37:0e:95:46:08:41:c9:7a:b8:89:f6:74:7f:84:
                        14:aa:3f:91:94:78:df:8d:b0:47:9b:1f:2e:88:67:
                        eb:13:71:a7:57:a8:c3:cd:fe:22:ea:05:35:42:6f:
                        33:f5:46:05:27:c1:16:06:8c:66:14:fc:44:d3:56:
                        ae:8f:60:1b:df:72:02:a0:cb:89:bc:50:56:5f:aa:
                        97:57:03:7e:71:bc:71:7d:f3:9c:53:ae:12:0e:56:
                        f0:46:45:e6:61:de:37:fa:40:b0:3b:fa:7e:a5:26:
                        b0:06:67:49:b0:c1:c0:a1:35:5c:3d:c1:1b:d7:79:
                        bf:c3:93:d3:e4:fa:72:06:ec:84:1f:0c:6a:d0:92:
                        33:f4:8a:22:bf:af:53:78:48:bd:3d:63:f5:78:d0:
                        08:bf:52:a1:ea:40:4e:74:9a:89:dc:7a:b0:dd:c7:
                        e3:fe:35:58:91:65:c9:d2:00:d3:e9:fb:34:1e:39:
                        77:ac:2a:49:79:53:37:aa:59:16:ce:8c:18:d6:99:
                        af:e7:8f:9b:dc:b7:33:7e:00:41:3a:73:c0:5e:d7:
                        92:63
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
                    DNS:192.168.22.80:9876, DNS:192.168.22.80:19876, IP Address:192.168.22.80
        Signature Algorithm: sha256WithRSAEncryption
            8f:f2:7e:60:69:de:80:85:7b:9a:3e:c8:8a:c7:ee:ee:66:29:
            d7:bc:5b:54:0d:ae:50:7a:1a:9e:86:2e:ba:77:51:53:d3:c5:
            64:e6:ca:1c:69:45:3c:db:1a:a8:cb:b4:7b:f7:1d:16:59:cb:
            3c:89:80:1b:89:a3:cf:f3:6d:08:40:83:6f:f5:f5:f3:87:1f:
            3c:0a:42:e3:c6:7c:56:be:5c:50:15:f5:2b:e7:cb:33:03:43:
            30:e9:30:81:91:66:d7:34:70:ac:7b:09:bf:b0:32:41:09:7d:
            0d:7d:ee:6a:19:4b:0e:18:a8:8d:0c:d7:3b:c7:c2:28:7a:c3:
            c8:4a:34:cb:2a:90:2a:70:af:03:04:3c:93:6b:ee:3f:c0:47:
            45:10:59:67:3d:36:50:29:30:38:b2:f7:ff:a7:06:b3:b5:3b:
            66:64:21:87:ef:ab:0e:e7:e7:4c:4d:06:5a:3b:4c:1e:0f:d4:
            de:08:6c:cc:24:6f:c1:1b:03:3d:27:4c:62:ea:a6:79:ab:f3:
            a7:13:8a:26:e3:2a:e7:c2:bf:28:f4:de:58:6d:e2:d7:ed:e6:
            bf:a7:b4:eb:61:c6:89:0c:df:fc:41:6d:b0:9a:ae:64:59:a8:
            4e:86:5d:11:7e:1d:f6:b1:df:98:02:97:d8:12:63:dc:bf:3a:
            10:5e:46:6c
    ```

## Create an X.509 certificate for your client application for TLS authentication

The client applications need a certificate that is issued by the same certification authority that the GREP11 server uses. The process is similar- create a private key, pass that as input for the creation of a certificte signing request, and then pass that to the certification authority in order to receive a certificate.  We used these commands:

``` bash
openssl genrsa -out client-key.pem 2048
```

``` bash
openssl req -new -key client-key.pem -out client.csr
```

``` bash
openssl x509 -req -days 1000 -in client.csr -CA ca.pem -CAcreateserial -CAkey ca.key -out client.pem
```

Here is a listing of the client certificate that you will be using in the lab. It will be configured properly for you so that your client program can present it to the GREP11 server:

``` bash
openssl x509 -in client.pem -noout -text
```

??? example "Example Output"

    ``` hl_lines="11"
    Certificate:
        Data:
            Version: 1 (0x0)
            Serial Number:
                4f:07:1f:6e:5e:09:33:8d:6f:39:52:01:36:06:3e:bb:28:50:c9:41
            Signature Algorithm: sha256WithRSAEncryption
            Issuer: C = US, ST = Virginia, L = Herndon, O = IBM, OU = IBM Washington Systems Center, CN = wschpvs
            Validity
                Not Before: Jul 23 21:52:13 2020 GMT
                Not After : Apr 19 21:52:13 2023 GMT
            Subject: C = US, ST = Virginia, L = Herndon, O = IBM, OU = IBM Washington Systems Center, CN = GREP11 Lab Students
            Subject Public Key Info:
                Public Key Algorithm: rsaEncryption
                    RSA Public-Key: (2048 bit)
                    Modulus:
                        00:c4:ab:d2:80:e2:38:b8:9d:6b:7f:f0:ae:09:72:
                        c4:64:57:1a:32:f9:eb:b5:31:e9:bf:81:0b:c1:d7:
                        01:b6:6d:c7:7c:26:d3:49:6b:3b:fd:54:11:cc:5e:
                        7e:2f:be:59:54:4d:a4:c4:5c:08:87:1f:2b:0f:bc:
                        cf:91:cd:21:a9:8e:bd:34:87:bb:3c:dd:e7:64:e4:
                        e2:5e:30:d1:27:74:d4:7e:9f:3b:2f:40:68:3d:d9:
                        7d:d7:ff:e8:b6:ad:3a:ab:ca:d9:74:51:fa:12:59:
                        a3:e1:2a:13:e2:20:34:01:6a:a3:6a:7c:36:be:81:
                        37:2a:cb:3b:85:a0:59:f1:60:ca:be:de:ce:d0:0d:
                        ab:31:e4:09:09:31:39:06:cb:8d:48:73:56:ec:4f:
                        93:cd:85:d9:59:89:9a:f3:d2:e5:d6:de:8e:e5:a5:
                        4d:20:89:07:e0:1b:bd:a0:09:6f:94:92:e7:77:f6:
                        76:f3:da:16:01:7c:80:76:cf:29:11:39:bc:6d:71:
                        4a:bd:08:ca:29:da:d0:16:22:f5:f0:4d:c2:83:e4:
                        8c:72:9b:b4:17:bf:48:09:29:6b:bd:4d:35:3b:44:
                        9f:2f:75:7f:e2:a6:08:e7:ae:bb:4d:77:10:83:e6:
                        cd:09:21:18:68:b2:ad:0a:f0:74:e5:47:ad:41:e5:
                        41:6d
                    Exponent: 65537 (0x10001)
        Signature Algorithm: sha256WithRSAEncryption
            3f:ce:22:e4:ba:6b:1d:68:aa:b4:07:35:22:0c:d7:55:e7:dc:
            4f:21:ca:17:ca:14:cd:b9:7e:ad:f1:93:84:9f:e4:96:84:10:
            39:d7:36:1f:66:2d:15:1b:40:96:a6:79:5e:e4:a2:00:c8:fa:
            fe:1a:42:9f:37:45:c4:c7:d9:21:c7:04:82:37:21:a0:41:ca:
            92:b7:b8:15:db:62:55:c8:55:02:d6:d7:3a:fb:1d:96:33:54:
            c8:f2:fd:d7:65:39:7c:12:13:e6:b5:91:fd:0e:68:21:30:41:
            82:f0:c0:74:a5:12:18:5e:e4:4b:83:1a:7f:a3:cc:f7:0a:aa:
            8f:f0:4e:0a:01:45:64:98:10:9a:f1:41:48:46:ec:df:3b:db:
            c5:2f:bb:ec:f7:dc:77:f0:98:e2:99:f4:20:fc:ff:45:24:af:
            8a:56:e7:60:2f:aa:cc:af:8a:ae:a8:1f:33:a0:8e:5e:3e:46:
            69:10:f0:b9:6c:02:6d:b3:4e:ef:cd:ac:06:86:67:9b:c9:44:
            73:00:37:61:06:df:32:ce:e5:c3:28:04:8f:c8:e4:81:ec:16:
            2c:dd:63:37:bd:07:ab:fc:f9:ac:55:e2:4a:b0:9b:5e:5d:f5:
            f1:89:90:0b:59:50:da:21:cb:db:96:c7:6d:fa:54:e4:f2:74:
            86:e2:71:6e
    ```

!!! note
    For the lab, each student will use a copy of the same client certificate. In most production use cases each client would use their own certificate which uniquely identifies them.

## List Crypto Domains on your Hyper Protect Virtual Servers LPAR

You can list your available domains with the Hyper Protect Virtual Server CLI:

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

I will show both files formats (YAML and JSON) first.  Then I will show both methods of starting the GREP11 server, and then explain the difference between the two start methods.

### YAML file for GREP11 server configuration

This is the YAML file for a GREP11 server that will listen for client connections on port 9876, and will use domain 22 (hex 16) of Crypto Express 7S card 08:

??? example "GREP11 server YAML configuration file"

    ```
    version: v1
    #
    # use this file with the 'hpvs deploy' command, e.g.,
    #
    #  hpvs deploy --config $HOME/hpvs/config/grep11/vs_grep11_80-9876.yml 
    #
    type: virtualserver
    virtualservers:
    - name: grep11-08-0016-9876
    host: wsclpar80
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
        value: "@/home/hyper-protect-lab/hpvs/config/grep11/keys/server80-9876-19876.pem"
    - key: EP11SERVER_EP11CRYPTO_CONNECTION_TLS_KEYFILEBYTES
        value: "@/home/hyper-protect-lab/hpvs/config/grep11/keys/server80-9876-19876-key.pem"
    - key: EP11SERVER_EP11CRYPTO_CONNECTION_TLS_CACERTBYTES
        value: "@/home/hyper-protect-lab/hpvs/config/grep11/keys/ca.pem"
    - key: EP11SERVER_EP11CRYPTO_CONNECTION_TLS_ENABLED
        value: "true"
    - key: EP11SERVER_EP11CRYPTO_CONNECTION_TLS_MUTUAL
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

    ```
    {
    "EP11SERVER_EP11CRYPTO_DOMAIN":"08.0016",
    "EP11SERVER_EP11CRYPTO_CONNECTION_TLS_CERTFILEBYTES":"@/home/hyper-protect-lab/hpvs/config/grep11/keys/server80-9876-19876.pem",
    "EP11SERVER_EP11CRYPTO_CONNECTION_TLS_KEYFILEBYTES":"@/home/hyper-protect-lab/hpvs/config/grep11/keys/server80-9876-19876-key.pem",
    "EP11SERVER_EP11CRYPTO_CONNECTION_TLS_CACERTBYTES":"@/home/hyper-protect-lab/hpvs/config/grep11/keys/ca.pem",
    "EP11SERVER_EP11CRYPTO_CONNECTION_TLS_ENABLED":true,
    "EP11SERVER_EP11CRYPTO_CONNECTION_TLS_MUTUAL":true,
    "TLS_GRPC_CERTS_DOMAIN_CRT":"\\n",
    "TLS_GRPC_CERTS_DOMAIN_KEY":"\\n",
    "TLS_GRPC_CERTS_ROOTCA_CRT":"\\n"
    }
    ```

## Start the GREP11 server

If you looked carefully at the JSON file and the YAML file in the previous section, you may have noticed that the YAML file contains more information than the JSON file contains.  As a result, the syntax is simpler for the command that uses the YAML file.  The command that uses the JSON file uses command line arguments to provide some of the information that the YAML file contained.  Compare the two methods:

???+ example "Command to start the GREP11 server with the YAML file"

    ```
    hpvs deploy --config $HOME/hpvs/config/grep11/vs_grep11_80-9876.yml
    ```

???+ example "Command to start the GREP11 server with the JSON file"

    ```
    hpvs vs create --name grep11-08-0016-9876 --repo hpcsKpGrep11_runq --tag 1.2.1 --crypto_matrix=08.0016 --cpu 2 --ram 2048 --envjsonpath ${HOME}/hpvs/config/grep11/grep11_env_08.0016.json --ports "{containerport = 9876, protocol = tcp, hostport = 9876}"
    ```

!!! note "Difference between the two commands"
    I like the simplicity of the `hpvs deploy`method much better than the long syntax of the `hpvs vs create` command.  But the `hpvs vs create` command has a benefit-  the `hpvs deploy` command uploads the GREP11 server Docker image from your workstation where you run the CLI to the Hyper Protect Virtual Server LPAR, every single time.  This Docker image only needs to be sent up once.  The `hpvs vs create` command is smart enough to have this figured out and not do the unnecessary upload of the Docker image the second and subsequent times you run it.

!!! Note
    Starting now, when you navigate to the next section of the lab, you should enter all the commands shown in the lab.  Only the commands in this section were for reference.
    
[^1]: The GREP11 server is a feature provided by Hyper Protect Virtual Servers. You are not required to use it. If you do not use this feature, you do not have to define Crypto Express domains to the LPAR.  You may wish to for other purposes, and you can use the other modes offered by Crypto Express cards for those purposes, but you must use EP11 mode for usage by the GREP11 server.
