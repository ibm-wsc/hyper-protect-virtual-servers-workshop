# How to set up a GREP11 server

## Hyper Protect Virtual Servers LPAR setup

Hyper Protect Virtual Servers runs in an LPAR that is defined in Secure Service Container (SSC) mode. Defining an LPAR is a normal task for an IBM Z or LinuxONE systems administrator, typically performed from the Hardware Management Console (HMC).

The systems administrator must[^1] dedicate one or more domains of one or more Crypto Express cards to the Hyper Protect Virtual Servers LPAR in order to use the GREP11 server. These Crypto Express cards must be defined in EP11 mode to your IBM Z or LinuxONE server in order to be used by a GREP11 server. 
These tasks are documented in the Hyper Protect Virtual Servers documentation, or in IBM publications referenced in the Hyper Protect Virtual Servers documentation. 

The version of Hyper Protect Virtual Servers that we will be using in this lab is actually a pre-release version of 1.2.1, expected to be available soon, and this lab will be updated to include a public link to the documentation when it becomes available.

A GREP11 server communicates with one and only one Crypto Express domain, and vice versa.  You can run multiple GREP11 servers if you have multiple domains configured to your Hyper Protect Virtual Servers LPAR.

## Overview of GREP11 server setup

This section provides an overview of what the remaining sections of this page will provide in detail.

The following steps are required:

1. Create a Certification Authority (CA) certificate and key

2. Create a GREP11 server X.509 certificate for TLS authentication

3. Create an X.509 certificate for your client application for TLS authentication

4. List Crypto Domains on your Hyper Protect Virtual Servers LPAR

5. Create YAML and/or JSON configuration files for GREP11 server initialization

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

What isn't commonly known, but is true, is that the RSA public key is contained within the private key.  It is for this reason that the private key is used as input to the following command, which will create a Certification Authority X.509 certificate. X.509 certificates contain information identifying the certificate holder, the attributes of the certificate, including what the certificate can be used for, and, importantly, the public key.  

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
    MIIDtTCCAp2gAwIBAgIUe4lZppOfAoaQvSSDJ4fayWJwyd0wDQYJKoZIhvcNAQEL
    BQAwajELMAkGA1UEBhMCVVMxETAPBgNVBAgMCFZpcmdpbmlhMRAwDgYDVQQHDAdI
    ZXJuZG9uMQwwCgYDVQQKDANJQk0xEDAOBgNVBAsMB0lCTSBXU0MxFjAUBgNVBAMM
    DTE5Mi4xNjguMjIuNzkwHhcNMjAwNjExMTczOTM0WhcNMjAwNzExMTczOTM0WjBq
    MQswCQYDVQQGEwJVUzERMA8GA1UECAwIVmlyZ2luaWExEDAOBgNVBAcMB0hlcm5k
    b24xDDAKBgNVBAoMA0lCTTEQMA4GA1UECwwHSUJNIFdTQzEWMBQGA1UEAwwNMTky
    LjE2OC4yMi43OTCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAMporCRt
    +PblqQ54xamzxn6XdOTYgIGdih1ifORB+jeBUNpXOcotcP7qZarafVyHL2EFEK1e
    xDmPz2G69hNtwx+V3YD1uCOuTmRQQvtqAp6PTj9b4hdKs4IXzmDUxmWP7xi+VbuP
    E9YsxBJpLk/uzhhfAFCWWqLI+/IA9qs/O/hMDaK5pn/lDlDvjR5Cqem8xmTAzVQ3
    qE6CYk4FOwyS2hNGN6rp7qCm1y7mD8igcu7yXyp7n6HGFCFn1FEZbJaRY7ls7kg6
    mZWEejsH6maSN7UCmNsrFCknYD/D0FcdU4uA4sRnghesJj2pgTHZOjQs7EoXwJdn
    l5tBbgesnUs3Gu8CAwEAAaNTMFEwHQYDVR0OBBYEFIZjwWSeH3dt9o614z6XFcMO
    Tt1LMB8GA1UdIwQYMBaAFIZjwWSeH3dt9o614z6XFcMOTt1LMA8GA1UdEwEB/wQF
    MAMBAf8wDQYJKoZIhvcNAQELBQADggEBAFJfCqrHxxRGYhK70GkjojWSY9MZ0vvi
    sTFUteBz5GBKxo3P1Pk3ZvniWJH/b9UyF36mEa/iTgI0o+ffcYvvqZr3qavxDbhi
    PgqNgAYPiXnPDJ4MStY5+6hClzycZroJqNJVeZbleNHIOA64LO+zwFpZXLxygZGx
    0xiU277HvtrO0ayWWCxHq9XNP4FPsrIw5m5RhvB3dFm6fxYSYTKbIV3vQtDNfMxW
    Ww7XtPnbwOm0BBkN1poL/jmeA/7lxUCGNr9IKh5PZtT1j+Laa/VhTlWt00qT6mAV
    861YN3vvo2U8F36omKKnYlCxojea0ECTRNp09B3CHEDBGKv6JdBXvtQ=
    -----END CERTIFICATE-----
    ```
The first and last lines are meant to assure you that this is a certificate, but the lines in between are less insightful. Fortunately the *openssl* utility comes to our rescue and allows us to print the certificate in geek- and human-readable form:

``` bash
openssl x509 -in ca.pem -text
```

??? example "Example output"

    ``` bash  hl_lines="36-39 41-42"
    Certificate:
        Data:
            Version: 3 (0x2)
            Serial Number:
                7b:89:59:a6:93:9f:02:86:90:bd:24:83:27:87:da:c9:62:70:c9:dd
            Signature Algorithm: sha256WithRSAEncryption
            Issuer: C = US, ST = Virginia, L = Herndon, O = IBM, OU = IBM WSC, CN = 192.168.22.79
            Validity
                Not Before: Jun 11 17:39:34 2020 GMT
                Not After : Jul 11 17:39:34 2020 GMT
            Subject: C = US, ST = Virginia, L = Herndon, O = IBM, OU = IBM WSC, CN = 192.168.22.79
            Subject Public Key Info:
                Public Key Algorithm: rsaEncryption
                    RSA Public-Key: (2048 bit)
                    Modulus:
                        00:ca:68:ac:24:6d:f8:f6:e5:a9:0e:78:c5:a9:b3:
                        c6:7e:97:74:e4:d8:80:81:9d:8a:1d:62:7c:e4:41:
                        fa:37:81:50:da:57:39:ca:2d:70:fe:ea:65:aa:da:
                        7d:5c:87:2f:61:05:10:ad:5e:c4:39:8f:cf:61:ba:
                        f6:13:6d:c3:1f:95:dd:80:f5:b8:23:ae:4e:64:50:
                        42:fb:6a:02:9e:8f:4e:3f:5b:e2:17:4a:b3:82:17:
                        ce:60:d4:c6:65:8f:ef:18:be:55:bb:8f:13:d6:2c:
                        c4:12:69:2e:4f:ee:ce:18:5f:00:50:96:5a:a2:c8:
                        fb:f2:00:f6:ab:3f:3b:f8:4c:0d:a2:b9:a6:7f:e5:
                        0e:50:ef:8d:1e:42:a9:e9:bc:c6:64:c0:cd:54:37:
                        a8:4e:82:62:4e:05:3b:0c:92:da:13:46:37:aa:e9:
                        ee:a0:a6:d7:2e:e6:0f:c8:a0:72:ee:f2:5f:2a:7b:
                        9f:a1:c6:14:21:67:d4:51:19:6c:96:91:63:b9:6c:
                        ee:48:3a:99:95:84:7a:3b:07:ea:66:92:37:b5:02:
                        98:db:2b:14:29:27:60:3f:c3:d0:57:1d:53:8b:80:
                        e2:c4:67:82:17:ac:26:3d:a9:81:31:d9:3a:34:2c:
                        ec:4a:17:c0:97:67:97:9b:41:6e:07:ac:9d:4b:37:
                        1a:ef
                    Exponent: 65537 (0x10001)
            X509v3 extensions:
                X509v3 Subject Key Identifier: 
                    86:63:C1:64:9E:1F:77:6D:F6:8E:B5:E3:3E:97:15:C3:0E:4E:DD:4B
                X509v3 Authority Key Identifier: 
                    keyid:86:63:C1:64:9E:1F:77:6D:F6:8E:B5:E3:3E:97:15:C3:0E:4E:DD:4B

                X509v3 Basic Constraints: critical
                    CA:TRUE
        Signature Algorithm: sha256WithRSAEncryption
            52:5f:0a:aa:c7:c7:14:46:62:12:bb:d0:69:23:a2:35:92:63:
            d3:19:d2:fb:e2:b1:31:54:b5:e0:73:e4:60:4a:c6:8d:cf:d4:
            f9:37:66:f9:e2:58:91:ff:6f:d5:32:17:7e:a6:11:af:e2:4e:
            02:34:a3:e7:df:71:8b:ef:a9:9a:f7:a9:ab:f1:0d:b8:62:3e:
            0a:8d:80:06:0f:89:79:cf:0c:9e:0c:4a:d6:39:fb:a8:42:97:
            3c:9c:66:ba:09:a8:d2:55:79:96:e5:78:d1:c8:38:0e:b8:2c:
            ef:b3:c0:5a:59:5c:bc:72:81:91:b1:d3:18:94:db:be:c7:be:
            da:ce:d1:ac:96:58:2c:47:ab:d5:cd:3f:81:4f:b2:b2:30:e6:
            6e:51:86:f0:77:74:59:ba:7f:16:12:61:32:9b:21:5d:ef:42:
            d0:cd:7c:cc:56:5b:0e:d7:b4:f9:db:c0:e9:b4:04:19:0d:d6:
            9a:0b:fe:39:9e:03:fe:e5:c5:40:86:36:bf:48:2a:1e:4f:66:
            d4:f5:8f:e2:da:6b:f5:61:4e:55:ad:d3:4a:93:ea:60:15:f3:
            ad:58:37:7b:ef:a3:65:3c:17:7e:a8:98:a2:a7:62:50:b1:a2:
            37:9a:d0:40:93:44:da:74:f4:1d:c2:1c:40:c1:18:ab:fa:25:
            d0:57:be:d4
    -----BEGIN CERTIFICATE-----
    MIIDtTCCAp2gAwIBAgIUe4lZppOfAoaQvSSDJ4fayWJwyd0wDQYJKoZIhvcNAQEL
    BQAwajELMAkGA1UEBhMCVVMxETAPBgNVBAgMCFZpcmdpbmlhMRAwDgYDVQQHDAdI
    ZXJuZG9uMQwwCgYDVQQKDANJQk0xEDAOBgNVBAsMB0lCTSBXU0MxFjAUBgNVBAMM
    DTE5Mi4xNjguMjIuNzkwHhcNMjAwNjExMTczOTM0WhcNMjAwNzExMTczOTM0WjBq
    MQswCQYDVQQGEwJVUzERMA8GA1UECAwIVmlyZ2luaWExEDAOBgNVBAcMB0hlcm5k
    b24xDDAKBgNVBAoMA0lCTTEQMA4GA1UECwwHSUJNIFdTQzEWMBQGA1UEAwwNMTky
    LjE2OC4yMi43OTCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAMporCRt
    +PblqQ54xamzxn6XdOTYgIGdih1ifORB+jeBUNpXOcotcP7qZarafVyHL2EFEK1e
    xDmPz2G69hNtwx+V3YD1uCOuTmRQQvtqAp6PTj9b4hdKs4IXzmDUxmWP7xi+VbuP
    E9YsxBJpLk/uzhhfAFCWWqLI+/IA9qs/O/hMDaK5pn/lDlDvjR5Cqem8xmTAzVQ3
    qE6CYk4FOwyS2hNGN6rp7qCm1y7mD8igcu7yXyp7n6HGFCFn1FEZbJaRY7ls7kg6
    mZWEejsH6maSN7UCmNsrFCknYD/D0FcdU4uA4sRnghesJj2pgTHZOjQs7EoXwJdn
    l5tBbgesnUs3Gu8CAwEAAaNTMFEwHQYDVR0OBBYEFIZjwWSeH3dt9o614z6XFcMO
    Tt1LMB8GA1UdIwQYMBaAFIZjwWSeH3dt9o614z6XFcMOTt1LMA8GA1UdEwEB/wQF
    MAMBAf8wDQYJKoZIhvcNAQELBQADggEBAFJfCqrHxxRGYhK70GkjojWSY9MZ0vvi
    sTFUteBz5GBKxo3P1Pk3ZvniWJH/b9UyF36mEa/iTgI0o+ffcYvvqZr3qavxDbhi
    PgqNgAYPiXnPDJ4MStY5+6hClzycZroJqNJVeZbleNHIOA64LO+zwFpZXLxygZGx
    0xiU277HvtrO0ayWWCxHq9XNP4FPsrIw5m5RhvB3dFm6fxYSYTKbIV3vQtDNfMxW
    Ww7XtPnbwOm0BBkN1poL/jmeA/7lxUCGNr9IKh5PZtT1j+Laa/VhTlWt00qT6mAV
    861YN3vvo2U8F36omKKnYlCxojea0ECTRNp09B3CHEDBGKv6JdBXvtQ=
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
openssl x509 -sha256 -req -in server80-9876-19876.csr -CA ca.pem -CAkey ca.key -set_serial 8086 -extfile openssl.cnf -extensions server -days 36500 -outform PEM -out server80-9876-19876.pem
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
            Issuer: C = US, ST = Virginia, L = Herndon, O = IBM, OU = IBM WSC, CN = 192.168.22.79
            Validity
                Not Before: Jun 30 19:45:06 2020 GMT
                Not After : Jun  6 19:45:06 2120 GMT
            Subject: C = US, ST = Virginia, L = Herndon, O = IBM, OU = IBM WSC, CN = 192.168.22.80
            Subject Public Key Info:
                Public Key Algorithm: rsaEncryption
                    RSA Public-Key: (2048 bit)
                    Modulus:
                        00:c2:66:77:9d:ef:38:ab:48:05:8d:ad:7d:aa:03:
                        b6:b1:23:d9:39:81:20:6b:d1:60:21:64:a2:b4:c3:
                        71:9d:c7:36:0b:12:f9:51:ea:75:62:bd:02:11:54:
                        7d:22:bd:38:75:2e:7b:77:05:4b:5e:63:a0:2d:55:
                        72:51:b7:28:03:0c:1a:72:8e:d3:b1:64:b2:40:4e:
                        25:61:ab:d7:fe:44:e7:d1:0d:5e:dd:19:d6:af:e2:
                        9e:1f:34:f9:44:c0:8c:c3:b0:e4:f3:43:4b:48:dd:
                        34:8c:82:25:01:0b:a0:ad:f7:80:92:f2:84:d9:55:
                        4d:e0:7d:85:78:b9:f6:de:e6:9f:61:cf:ff:b1:ed:
                        93:24:06:e7:51:fd:ae:c0:7f:f4:49:61:ab:c8:c8:
                        46:1d:da:f2:76:97:f8:87:4f:a3:3c:c4:c4:46:00:
                        83:77:11:86:78:4f:cb:c2:b7:7d:18:fc:15:10:78:
                        90:10:96:3e:9c:cb:1f:ce:c5:1d:4f:88:72:11:88:
                        36:8e:42:34:7f:a0:51:0c:82:ee:d5:46:d4:b3:f5:
                        7d:d0:53:6f:22:18:01:d5:08:fa:73:12:6c:02:d6:
                        9f:53:c9:23:e7:46:c9:21:7e:66:3e:e0:01:44:f8:
                        34:e4:d8:37:30:ff:f0:83:a2:ba:a3:fb:ec:cd:01:
                        89:43
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
            c1:64:4d:f9:81:66:0f:a8:44:63:8f:1c:82:3b:1e:05:45:b2:
            51:5f:b2:8a:cd:e3:e9:5b:1e:83:e8:a9:54:21:01:57:20:49:
            b6:6a:e2:a4:58:24:44:73:5c:e4:81:88:57:e1:d9:ca:ef:2c:
            d9:b3:20:05:e4:0d:e9:46:22:3f:82:27:1e:b5:8a:7b:84:ef:
            2a:af:b2:78:58:c2:a2:37:fa:dc:8d:75:4b:09:6d:67:b7:eb:
            f4:91:fc:2b:14:ef:94:75:1e:93:f1:7b:5d:69:23:8e:de:1e:
            69:9e:a8:b9:62:47:28:ab:18:6a:00:0b:9f:b6:5e:cf:62:74:
            57:df:1f:15:a0:81:4f:21:bc:c3:08:05:c1:c1:8d:a1:fa:78:
            3d:e8:be:b8:eb:03:9c:3e:fa:8a:7d:70:09:56:47:4b:20:ea:
            02:3e:85:dc:a0:af:81:77:c9:1d:91:a2:68:b7:9c:c4:8f:a1:
            dc:48:16:4f:38:5d:c7:d3:dc:51:b0:ff:a4:1b:aa:71:fe:34:
            b0:59:04:43:a6:88:69:78:41:56:70:cb:7b:e1:2a:55:e1:1b:
            f0:2a:11:25:ac:58:81:15:b0:9b:7f:f0:45:3f:ef:70:05:ce:
            81:4f:3e:9f:17:4a:70:f2:65:65:3d:52:7e:af:59:46:d3:f8:
            44:71:e6:c9

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
