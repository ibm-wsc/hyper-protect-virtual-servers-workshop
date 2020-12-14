# Exercise 1 - List Mechanisms

## Explanation of Mechanisms

*Mechanisms* is the PKCS #11 standard's fancy word for *algorithms*.  

In the introduction of the *PKCS#11 Cryptographic Token Interface Base Specification 2.40* document, there is this statement: *Details of cryptographic mechanisms (algorithms) may be found in the associated PKCS#11 Mechanisms documents.*

And then in the *PKCS#11 Cryptographic Token Interface Current Mechanisms Specification Version 2.40* it says `A mechanism specifies precisely how a certain cryptographic process is to be performed.`

This exercise will list the mechanisms supported by the Crypto Express card in EP11 mode.

The program that you will invoke in this exercise issues a *GetMechanismList()* function call to the GREP11 server, and then it will iterate through the list returned, calling *GetMechanismInfo()* against each mechanism to obtain more info about it. 

Run the program and then a brief discussion of the results will follow:

1. You may wish to view the source code used by the lab exercises while running them.  It is handy to expand the *Outline* view in the left pane of Visual Studio Code which will show the various function and type definitions of the source code you have highlighted in the main portion of the Visual Studio Code window.  The Outline view may not be necessary for smaller source code files but it comes in handy for some of the larger source code files. The screen snippet below shows where the Outline view is:

    ![Expand outling](images/grep11-0070_expand-outline.png)


2. Change to the lab directory in your terminal:

    ``` bash
    cd ../lab
    ```

3. In the prior section, the *go test* command built the executable file it needed transparently to us. For our lab exercises, we need to build the executable binary.  We will use a single executable for all of the exercises, and invoke the various exercises through command line arguments. Build the executable now with this command:

    ``` bash
    go build
    ```
   
    !!! note
        The absence of any messages is your indication of success!

4. The `go build` command should have created an excutable file that shares the name of the directory in which you are working- `lab`. List that file and you should see it has a recent date and timestamp.

    ``` bash
    ls -l lab
    ```

    ???+ example "Example output"
    
        ``` bash
        -rwxr-xr-x 1 hyper-protect-lab hyper-protect-lab 17545825 Jul  8 06:26 lab
        ```

    !!! tip
        You may be doing this lab in a virtual machine whose clock is set for a time zone different from where you are. You can always issue the `date` command to see what time your virtual machine thinks it is at the moment.

5. Run the first exercise by passing the appropriate argument, shown below, to the *lab* program:

    ``` bash
    ./lab --ex1
    ```

    ??? example "Example output"

        ```
        Got mechanism list of length 94:

        Mechanism  0 : CKM_RSA_PKCS 
        MinKeySize:512 MaxKeySize:4096 Flags:404224  
        
        Mechanism  1 : CKM_RSA_PKCS_OAEP 
        MinKeySize:512 MaxKeySize:4096 Flags:393984  
        
        Mechanism  2 : CKM_RSA_PKCS_KEY_PAIR_GEN 
        MinKeySize:512 MaxKeySize:4096 Flags:65536  
        
        Mechanism  3 : CKM_RSA_X9_31_KEY_PAIR_GEN 
        MinKeySize:512 MaxKeySize:4096 Flags:65536  
        
        Mechanism  4 : CKM_RSA_PKCS_PSS 
        MinKeySize:512 MaxKeySize:4096 Flags:10240  
        
        Mechanism  5 : CKM_SHA1_RSA_X9_31 
        MinKeySize:512 MaxKeySize:4096 Flags:10240  
        
        Mechanism  6 : CKM_SHA1_RSA_PKCS 
        MinKeySize:512 MaxKeySize:4096 Flags:10240  
        
        Mechanism  7 : CKM_SHA1_RSA_PKCS_PSS 
        MinKeySize:512 MaxKeySize:4096 Flags:10240  
        
        Mechanism  8 : CKM_SHA256_RSA_PKCS 
        MinKeySize:512 MaxKeySize:4096 Flags:10240  
        
        Mechanism  9 : CKM_SHA256_RSA_PKCS_PSS 
        MinKeySize:512 MaxKeySize:4096 Flags:10240  
        
        Mechanism  10 : CKM_SHA224_RSA_PKCS 
        MinKeySize:512 MaxKeySize:4096 Flags:10240  
        
        Mechanism  11 : CKM_SHA224_RSA_PKCS_PSS 
        MinKeySize:512 MaxKeySize:4096 Flags:10240  
        
        Mechanism  12 : CKM_SHA384_RSA_PKCS 
        MinKeySize:512 MaxKeySize:4096 Flags:10240  
        
        Mechanism  13 : CKM_SHA384_RSA_PKCS_PSS 
        MinKeySize:512 MaxKeySize:4096 Flags:10240  
        
        Mechanism  14 : CKM_SHA512_RSA_PKCS 
        MinKeySize:512 MaxKeySize:4096 Flags:10240  
        
        Mechanism  15 : CKM_SHA512_RSA_PKCS_PSS 
        MinKeySize:512 MaxKeySize:4096 Flags:10240  
        
        Mechanism  16 : CKM_AES_KEY_GEN 
        MinKeySize:16 MaxKeySize:32 Flags:32768  
        
        Mechanism  17 : CKM_AES_ECB 
        MinKeySize:16 MaxKeySize:32 Flags:768  
        
        Mechanism  18 : CKM_AES_CBC 
        MinKeySize:16 MaxKeySize:32 Flags:393984  
        
        Mechanism  19 : CKM_AES_CBC_PAD 
        MinKeySize:16 MaxKeySize:32 Flags:393984  
        
        Mechanism  20 : CKM_DES2_KEY_GEN 
        MinKeySize:16 MaxKeySize:16 Flags:32768  
        
        Mechanism  21 : CKM_DES3_KEY_GEN 
        MinKeySize:24 MaxKeySize:24 Flags:32768  
        
        Mechanism  22 : CKM_DES3_ECB 
        MinKeySize:16 MaxKeySize:24 Flags:768  
        
        Mechanism  23 : CKM_DES3_CBC 
        MinKeySize:16 MaxKeySize:24 Flags:393984  
        
        Mechanism  24 : CKM_DES3_CBC_PAD 
        MinKeySize:16 MaxKeySize:24 Flags:393984  
        
        Mechanism  25 : CKM_GENERIC_SECRET_KEY_GEN 
        MinKeySize:8 MaxKeySize:256 Flags:557056  
        
        Mechanism  26 : CKM_SHA256 
        Flags:1024  
        
        Mechanism  27 : CKM_SHA256_KEY_DERIVATION 
        Flags:524288  
        
        Mechanism  28 : CKM_SHA256_HMAC 
        MinKeySize:16 MaxKeySize:32 Flags:10240  
        
        Mechanism  29 : CKM_SHA224 
        Flags:1024  
        
        Mechanism  30 : CKM_SHA224_KEY_DERIVATION 
        Flags:524288  
        
        Mechanism  31 : CKM_SHA224_HMAC 
        MinKeySize:14 MaxKeySize:32 Flags:10240  
        
        Mechanism  32 : CKM_SHA_1 
        Flags:1024  
        
        Mechanism  33 : CKM_SHA1_KEY_DERIVATION 
        Flags:524288  
        
        Mechanism  34 : CKM_SHA_1_HMAC 
        MinKeySize:10 MaxKeySize:32 Flags:10240  
        
        Mechanism  35 : CKM_SHA384 
        Flags:1024  
        
        Mechanism  36 : CKM_SHA384_KEY_DERIVATION 
        Flags:524288  
        
        Mechanism  37 : CKM_SHA384_HMAC 
        MinKeySize:24 MaxKeySize:32 Flags:10240  
        
        Mechanism  38 : CKM_SHA512 
        Flags:1024  
        
        Mechanism  39 : CKM_SHA512_KEY_DERIVATION 
        Flags:524288  
        
        Mechanism  40 : CKM_SHA512_HMAC 
        MinKeySize:32 MaxKeySize:32 Flags:10240  
        
        Mechanism  41 : CKM_SHA512_256 
        Flags:1024  
        
        Mechanism  42 : CKM_IBM_SHA512_256 
        Flags:1024  
        
        Mechanism  43 : CKM_SHA512_256_HMAC 
        MinKeySize:16 MaxKeySize:32 Flags:10240  
        
        Mechanism  44 : CKM_IBM_SHA512_256_HMAC 
        MinKeySize:16 MaxKeySize:32 Flags:10240  
        
        Mechanism  45 : CKM_SHA512_224 
        Flags:1024  
        
        Mechanism  46 : CKM_IBM_SHA512_224 
        Flags:1024  
        
        Mechanism  47 : CKM_SHA512_224_HMAC 
        MinKeySize:14 MaxKeySize:32 Flags:10240  
        
        Mechanism  48 : CKM_IBM_SHA512_224_HMAC 
        MinKeySize:14 MaxKeySize:32 Flags:10240  
        
        Mechanism  49 : CKM_EC_KEY_PAIR_GEN 
        MinKeySize:192 MaxKeySize:521 Flags:26279936  
        
        Mechanism  50 : CKM_ECDSA 
        MinKeySize:192 MaxKeySize:521 Flags:26224640  
        
        Mechanism  51 : CKM_ECDSA_SHA1 
        MinKeySize:192 MaxKeySize:521 Flags:26224640  
        
        Mechanism  52 : CKM_ECDH1_DERIVE 
        MinKeySize:192 MaxKeySize:521 Flags:18350080  
        
        Mechanism  53 : CKM_IBM_ECDH1_DERIVE_RAW 
        MinKeySize:192 MaxKeySize:521 Flags:18350080  
        
        Mechanism  54 : CKM_IBM_EC_MULTIPLY 
        MinKeySize:192 MaxKeySize:521 Flags:256  
        
        Mechanism  55 : CKM_DSA_PARAMETER_GEN 
        MinKeySize:1024 MaxKeySize:3072 Flags:32768  
        
        Mechanism  56 : CKM_DSA_KEY_PAIR_GEN 
        MinKeySize:1024 MaxKeySize:3072 Flags:65536  
        
        Mechanism  57 : CKM_DSA 
        MinKeySize:1024 MaxKeySize:3072 Flags:10240  
        
        Mechanism  58 : CKM_DSA_SHA1 
        MinKeySize:1024 MaxKeySize:3072 Flags:10240  
        
        Mechanism  59 : CKM_DH_PKCS_PARAMETER_GEN 
        MinKeySize:1024 MaxKeySize:3072 Flags:32768  
        
        Mechanism  60 : CKM_DH_PKCS_KEY_PAIR_GEN 
        MinKeySize:1024 MaxKeySize:3072 Flags:65536  
        
        Mechanism  61 : CKM_DH_PKCS_DERIVE 
        MinKeySize:1024 MaxKeySize:3072 Flags:524288  
        
        Mechanism  62 : CKM_IBM_DH_PKCS_DERIVE_RAW 
        MinKeySize:1024 MaxKeySize:3072 Flags:524288  
        
        Mechanism  63 : Mechanism(0x80010023) 
        MinKeySize:256 MaxKeySize:256 Flags:75776  
        
        Mechanism  64 : CKM_RSA_X9_31 
        MinKeySize:512 MaxKeySize:4096 Flags:10240  
        
        Mechanism  65 : CKM_PBE_SHA1_DES3_EDE_CBC 
        MinKeySize:24 MaxKeySize:24 Flags:32768  
        
        Mechanism  66 : CKM_IBM_SHA3_224 
        Flags:1024  
        
        Mechanism  67 : CKM_IBM_SHA3_256 
        Flags:1024  
        
        Mechanism  68 : CKM_IBM_SHA3_384 
        Flags:1024  
        
        Mechanism  69 : CKM_IBM_SHA3_512 
        Flags:1024  
        
        Mechanism  70 : Mechanism(0x80010025) 
        MinKeySize:14 MaxKeySize:32 Flags:10240  
        
        Mechanism  71 : Mechanism(0x80010026) 
        MinKeySize:16 MaxKeySize:32 Flags:10240  
        
        Mechanism  72 : Mechanism(0x80010027) 
        MinKeySize:24 MaxKeySize:32 Flags:10240  
        
        Mechanism  73 : Mechanism(0x80010028) 
        MinKeySize:32 MaxKeySize:32 Flags:10240  
        
        Mechanism  74 : CKM_IBM_ATTRIBUTEBOUND_WRAP 
        MaxKeySize:4096 Flags:393216  
        
        Mechanism  75 : Mechanism(0x1043) 
        MinKeySize:192 MaxKeySize:521 Flags:26224640  
        
        Mechanism  76 : CKM_IBM_ECDSA_SHA224 
        MinKeySize:192 MaxKeySize:521 Flags:26224640  
        
        Mechanism  77 : Mechanism(0x1044) 
        MinKeySize:192 MaxKeySize:521 Flags:26224640  
        
        Mechanism  78 : CKM_IBM_ECDSA_SHA256 
        MinKeySize:192 MaxKeySize:521 Flags:26224640  
        
        Mechanism  79 : Mechanism(0x1045) 
        MinKeySize:192 MaxKeySize:521 Flags:26224640  
        
        Mechanism  80 : CKM_IBM_ECDSA_SHA384 
        MinKeySize:192 MaxKeySize:521 Flags:26224640  
        
        Mechanism  81 : Mechanism(0x1046) 
        MinKeySize:192 MaxKeySize:521 Flags:26224640  
        
        Mechanism  82 : CKM_IBM_ECDSA_SHA512 
        MinKeySize:192 MaxKeySize:521 Flags:26224640  
        
        Mechanism  83 : Mechanism(0x80010031) 
        MinKeySize:192 MaxKeySize:521 Flags:26224640  
        
        Mechanism  84 : CKM_IBM_EC_C25519 
        MinKeySize:256 MaxKeySize:256 Flags:18350080  
        
        Mechanism  85 : CKM_IBM_EC_C448 
        MinKeySize:448 MaxKeySize:448 Flags:18350080  
        
        Mechanism  86 : CKM_IBM_EDDSA_SHA512 
        MinKeySize:256 MaxKeySize:256 Flags:17836032  
        
        Mechanism  87 : CKM_IBM_ED448_SHA3 
        MinKeySize:448 MaxKeySize:448 Flags:17836032  
        
        Mechanism  88 : CKM_IBM_EAC 
        Flags:524288  
        
        Mechanism  89 : CKM_IBM_RETAINKEY 
        MaxKeySize:256 Flags:131072  
        
        Mechanism  90 : CKM_IBM_CMAC 
        MinKeySize:16 MaxKeySize:32 Flags:10240  
        
        Mechanism  91 : CKM_AES_CMAC 
        MinKeySize:16 MaxKeySize:32 Flags:10240  
        
        Mechanism  92 : CKM_DES3_CMAC 
        MinKeySize:16 MaxKeySize:24 Flags:10240  
        
        Mechanism  93 : Mechanism(0x80070001) 
        MinKeySize:16 MaxKeySize:64 Flags:524288  
        ```

## Explanation of exercise output

Near the top of the output, you see the line:

`Got mechanism list of length 94:` 

This indicates there were 94 mechanisms returned by the call to *GetMechanismList()*.

Then, for each mechanism, two lines of output are created, one with the mechanism name, which usually start with `CKM_`.  The mechanism number has no significance, this is just an index number assigned by our lab program, starting at 0 and incremented by the program as it iterates over the list.

The information returned by *GetMechanismInfo()* includes the minimum key size and maximum key size for the mechanism, if applicable, and flags that are associated with the mechanism.  The numbers shown are in decimal, and the key sizes shown are in bytes.

!!! Tip
    If you are doing this lab at your leisure on your own system, you could look in *golang/ep11/header_consts.go* and find the flags within lines 596 through 649 of that file.  You'd have to convert the decimal value shown to hexadecimal.  If you are taking this lab as part of a virtual event, do not take the time now to deconstruct these flags-  you can always do that later since this lab is publicly available.

Observe that the large majority of mechanisms in the list have a name starting with *CKM_*, while a handful, such as item 77, 79, 81, and others, do not.  It appears that if the mechanism is not listed in the *golang/ep11/header_consts.go* file, within lines 205 to 594, then it will not have a name starting with *CKM_*.

Note that the PKCS #11 specification does not mandate that an implementation support all defined mechanisms. Even within the subset of the 94 mechanisms that are supported by IBM's EP11 library, some are for algorithms that are considered rather outdated, and, in some cases, downright insecure.  For example, the SHA-1 cryptographic hash algorithm is considered insecure these days, so any mechanims in the list that have SHA1 or SHA_1 in their name (and there are some) should be avoided.

Let's move on to Exercise 2.


