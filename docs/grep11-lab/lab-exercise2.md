# Exercise 2 - Encrypt and Decrypt Data

## Overview of Exercise 2

Exercise 2 creates an AES secret key and uses it to encrypt some *plaintext* data. Plaintext which has been encrypted is referred to as *ciphertext*.
This ciphertext is then decrypted back into plaintext using the same AES secret key that was used for encryption. The original plaintext is compared to- and is expected to be identical to- the plaintext that resulted from the encryption and decryption.

!!! note
    Plaintext data is often, but not necessarily, human-readable.  It is any data that could be of value to an adversary, if the adversary could obtain the data in its original, or plaintext, form.  Hence the need for encryption. If an adversary were to obtain ciphertext, it is unreadable and thus of little or no value to the adversary.

!!! note
    The PKCS #11 standard uses the phrase *secret key* often. *Secret key* is synonoymous with the term *symmetric key*, which is also a term commonly used in cryptography.  That is, a single key is used for both encryption and decryption.


1.  Run the second exercise:

    ``` bash
    ./lab --ex2
    ```

    ??? example "Example Output"

        ``` bash  linenums="1" hl_lines="1 23"
        Generated AES Key with mechanism  CKM_AES_KEY_GEN

        length of key blob is 256 bytes
        Key blob is (values in decimal):
        [0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 3 176 130 49 65 141 152 246 85 132 150 183 226 212 36 40 0 0 0 0 0 0 141 38 0 0 0 0 0 0 0 1 18 52 217 158 52 159 29 220 10 126 250 240 129 168 243 15 158 220 32 30 158 254 73 111 45 102 230 162 209 246 93 216 200 108 223 116 32 109 1 82 87 166 108 227 20 9 54 59 24 126 154 14 159 250 161 203 128 27 237 27 218 243 169 235 28 112 101 109 177 94 191 251 198 126 102 93 236 51 190 157 4 188 128 1 114 120 135 191 22 7 42 191 209 135 168 241 228 137 121 180 140 113 211 159 26 192 70 74 130 25 180 75 86 230 207 240 101 164 65 179 122 59 202 213 142 244 54 151 158 46 72 52 249 87 219 11 202 69 150 45 195 200 84 134 30 148 21 70 144 235 236 141 234 135 4 120 1 180 69 84 35 73 16 71 208 212 143 135 222 115 85 26 25 172 2 202 15 2 249 109 71 41 255 177 105 20 134 33 10 54 254 210]

        The above structure is what you would save in order to persist this key blob.

        Below is how the above structure would typically be saved, in PEM Format:

        -----BEGIN HSM ENCRYPTED AES SECRET KEY-----
        AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAADsIIxQY2Y9lWElrfi1CQo
        AAAAAAAAjSYAAAAAAAAAARI02Z40nx3cCn768IGo8w+e3CAenv5Jby1m5qLR9l3Y
        yGzfdCBtAVJXpmzjFAk2Oxh+mg6f+qHLgBvtG9rzqesccGVtsV6/+8Z+Zl3sM76d
        BLyAAXJ4h78WByq/0Yeo8eSJebSMcdOfGsBGSoIZtEtW5s/wZaRBs3o7ytWO9DaX
        ni5INPlX2wvKRZYtw8hUhh6UFUaQ6+yN6ocEeAG0RVQjSRBH0NSPh95zVRoZrALK
        DwL5bUcp/7FpFIYhCjb+0g==
        -----END HSM ENCRYPTED AES SECRET KEY-----

        WK virtualization mask is [0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0]
        See 6.2.2 of page 179 of http://public.dhe.ibm.com/security/cryptocards/pciecc4/EP11/docs/ep11-structure.pdf

        WK ID is [3 176 130 49 65 141 152 246 85 132 150 183 226 212 36 40]
        See 6.7.1 on page 182 of  http://public.dhe.ibm.com/security/cryptocards/pciecc4/EP11/docs/ep11-structure.pdf

        Blob version is [18 52]
        See 3.1.1 on page 141 of http://public.dhe.ibm.com/security/cryptocards/pciecc4/EP11/docs/ep11-structure.pdf

        Initialization Vector is [217 158 52 159 29 220 10 126 250 240 129 168 243 15]

        Encrypted part is [158 220 32 30 158 254 73 111 45 102 230 162 209 246 93 216 200 108 223 116 32 109 1 82 87 166 108 227 20 9 54 59 24 126 154 14 159 250 161 203 128 27 237 27 218 243 169 235 28 112 101 109 177 94 191 251 198 126 102 93 236 51 190 157 4 188 128 1 114 120 135 191 22 7 42 191 209 135 168 241 228 137 121 180 140 113 211 159 26 192 70 74 130 25 180 75 86 230 207 240 101 164 65 179 122 59 202 213 142 244 54 151 158 46 72 52 249 87 219 11 202 69 150 45 195 200 84 134 30 148 21 70 144 235 236 141 234 135 4 120 1 180 69 84]

        MAC is [35 73 16 71 208 212 143 135 222 115 85 26 25 172 2 202 15 2 249 109 71 41 255 177 105 20 134 33 10 54 254 210]
        MAC is 32 bytes, see field 15 in 3.1 on page 140 of http://public.dhe.ibm.com/security/cryptocards/pciecc4/EP11/docs/ep11-structure.pdf

        Checksum is [144 147 65], length of checksum is 3


        Original message, prior to any encryption or decryption operations:  Hello, this is a very long and creative message without any imagination

        in progress ciphertext is:
        ��!E�S���}�p�!��

        in progress ciphertext is:
        ��!E�S���}�p�!��7I�Hd�i�7p�/N˽.�g��K�;����_����NØtk���YH�^�

        Final ciphertext is:
        ��!E�S���}�p�!��7I�Hd�i�7p�/N˽.�g��K�;����_����NØtk���YH�^���H�5f�!��>-c

        In progress decryption is:


        In progress decryption is:
        Hello, this is a very long and creative message without any imag

        Final decryption is:
        Hello, this is a very long and creative message without any imagination

        Original message equals decrypted message!

        Original message:  Hello, this is a very long and creative message without any imagination
        Length of original message: 71

        Encrypted message
        ��!E�S���}�p�!��7I�Hd�i�7p�/N˽.�g��K�;����_����NØtk���YH�^���H�5f�!��>-c
        Length of encrypted message: 80

        Decrypted message: Hello, this is a very long and creative message without any imagination
        Length of decrypted message: 71
        ```

    The mechanism used to generate the key is *CKM_AES_KEY_GEN* which informs us that we wish to generate an AES secret key.

    The length of the key blob is 256 bytes.  Notice that this key blob contains metadata used by the EP11 library, so it contains much more than the actual key, which is 256 bits, not bytes.  In fact we can't even see the key in cleartext form.  It is encrypted within the key blob by the Crypto Express 7S card domain's _root wrapping key_. The only place the key we created is ever available in the clear is while it is being used inside the Crypto Express 7S card, because the root wrapping key never leaves the card.

    The ID of the root wrapping key of the Crypto Express 7S card domain is listed in the output in the line that starts with *WK ID is*.  This ID is the first sixteen bytes of the hash of the key. It is not the full hash, and even if it was, you cannot retrieve the original data from the hash, so the root wrapping key remains secret.

    The ID of the root wrapping key for our Crypto Express 7S card domain is

    ``` bash
    [3 176 130 49 65 141 152 246 85 132 150 183 226 212 36 40]
    ```

    As you perform the exercises in this lab, you may notice that any time I print out the *WK ID*, it will always be this value.

    The encryption in this exercise takes place in multiple calls.  One may either encrypt all of the data in a single call to a function named *Encrypt()*, or may make multiple calls by performing one or more calls to *EncryptUpdate()* followed by a call to *EncryptFinal()*.  This sample program uses the latter approach, so in the output you see two lines that say *In progress ciphertext is* that show the results so far after each *EncryptUpdate()* call and then a line that says *Final ciphertext is* after the call to *EncryptFinal()*.  Decryption follows the same pattern-  two calls to *DecryptUpdate()* followed by a call to *DecryptFinal()*, and the output in the program indicates this.

2. The program for this exercise defaults to a key length of 256 bits.  Try running with a key length of 128 bits:

    ``` bash
    ./lab --ex2 --keyLength 128
    ```

    ??? example "Example Output"

        ```
        Generated AES Key with mechanism  CKM_AES_KEY_GEN

        length of key blob is 256 bytes
        Key blob is (values in decimal):
        [0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 3 176 130 49 65 141 152 246 85 132 150 183 226 212 36 40 0 0 0 0 0 0 141 38 0 0 0 0 0 0 0 1 18 52 3 221 177 141 112 242 216 176 144 18 6 175 10 206 202 162 226 99 67 216 88 101 201 3 114 172 228 151 134 15 141 183 46 209 125 119 136 246 196 223 189 56 243 204 100 151 226 149 160 128 229 255 216 226 133 171 65 244 16 197 245 172 136 232 118 184 206 173 34 79 215 214 202 250 8 237 80 168 235 252 137 221 202 76 92 21 176 127 119 178 212 220 235 169 231 144 203 67 92 32 243 99 83 171 234 198 118 71 201 176 223 202 214 117 107 152 139 105 99 39 22 243 21 20 173 175 57 84 114 199 115 225 125 30 230 123 50 120 157 37 32 151 24 77 118 170 12 222 17 3 53 7 177 238 226 18 192 102 80 131 223 195 188 205 133 31 172 138 123 48 4 244 81 224 205 165 108 128 163 226 55 208 117 70 166 128 111 166 213 158]

        The above structure is what you would save in order to persist this key blob.

        Below is how the above structure would typically be saved, in PEM Format:

        -----BEGIN HSM ENCRYPTED AES SECRET KEY-----
        AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAADsIIxQY2Y9lWElrfi1CQo
        AAAAAAAAjSYAAAAAAAAAARI0A92xjXDy2LCQEgavCs7KouJjQ9hYZckDcqzkl4YP
        jbcu0X13iPbE370488xkl+KVoIDl/9jihatB9BDF9ayI6Ha4zq0iT9fWyvoI7VCo
        6/yJ3cpMXBWwf3ey1NzrqeeQy0NcIPNjU6vqxnZHybDfytZ1a5iLaWMnFvMVFK2v
        OVRyx3PhfR7mezJ4nSUglxhNdqoM3hEDNQex7uISwGZQg9/DvM2FH6yKezAE9FHg
        zaVsgKPiN9B1RqaAb6bVng==
        -----END HSM ENCRYPTED AES SECRET KEY-----

        WK virtualization mask is [0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0]
        See 6.2.2 of page 179 of http://public.dhe.ibm.com/security/cryptocards/pciecc4/EP11/docs/ep11-structure.pdf

        WK ID is [3 176 130 49 65 141 152 246 85 132 150 183 226 212 36 40]
        See 6.7.1 on page 182 of  http://public.dhe.ibm.com/security/cryptocards/pciecc4/EP11/docs/ep11-structure.pdf

        Blob version is [18 52]
        See 3.1.1 on page 141 of http://public.dhe.ibm.com/security/cryptocards/pciecc4/EP11/docs/ep11-structure.pdf

        Initialization Vector is [3 221 177 141 112 242 216 176 144 18 6 175 10 206]

        Encrypted part is [202 162 226 99 67 216 88 101 201 3 114 172 228 151 134 15 141 183 46 209 125 119 136 246 196 223 189 56 243 204 100 151 226 149 160 128 229 255 216 226 133 171 65 244 16 197 245 172 136 232 118 184 206 173 34 79 215 214 202 250 8 237 80 168 235 252 137 221 202 76 92 21 176 127 119 178 212 220 235 169 231 144 203 67 92 32 243 99 83 171 234 198 118 71 201 176 223 202 214 117 107 152 139 105 99 39 22 243 21 20 173 175 57 84 114 199 115 225 125 30 230 123 50 120 157 37 32 151 24 77 118 170 12 222 17 3 53 7 177 238 226 18 192 102]

        MAC is [80 131 223 195 188 205 133 31 172 138 123 48 4 244 81 224 205 165 108 128 163 226 55 208 117 70 166 128 111 166 213 158]
        MAC is 32 bytes, see field 15 in 3.1 on page 140 of http://public.dhe.ibm.com/security/cryptocards/pciecc4/EP11/docs/ep11-structure.pdf

        Checksum is [132 58 194], length of checksum is 3


        Original message, prior to any encryption or decryption operations:  Hello, this is a very long and creative message without any imagination

        in progress ciphertext is:
        ���[�F�Vl

        in progress ciphertext is:
        ���[̶:���l
                �.�W�+�]n,�#�-��{������4��uj��!�
                                                    v

        Final ciphertext is:
        ���[̶:���l
                �.�W�+�]n,�#�-��{������4��uj��!�
                                                    v��7Ѵ��,��5-��

        In progress decryption is:


        In progress decryption is:
        Hello, this is a very long and creative message without any imag

        Final decryption is:
        Hello, this is a very long and creative message without any imagination

        Original message equals decrypted message!

        Original message:  Hello, this is a very long and creative message without any imagination
        Length of original message: 71

        Encrypted message
        ���[̶:���l
                �.�W�+�]n,�#�-��{������4��uj��!�
                                                    v��7Ѵ��,��5-��
        Length of encrypted message: 80

        Decrypted message: Hello, this is a very long and creative message without any imagination
        Length of decrypted message: 71
        ```

    Our first run of this program used a key length of 256 bits, and this run used a key length of 128 bits.  Notice that the key blob itself is still 256 bytes in length. 

    By the way, the gibberish printed out in the encrypted message portion of the output is because a byte of ciphertext is likely to contain any value from 0 to 255 and only a portion of those will turn out to be printable characters if you try to treat it as if it was a character string and try to print it out like this program does-  you may see one or more lines, if one of the ciphertext bytes produced happens to be the value of a linefeed character, and if it is a carriage return without a line feed it may have "overwritten" some of the preceding gibberish.  Real life use cases aren't typically going to require you to print ciphertext as this program did for education purposes.

3. Try with a key length of 192 bits:

    ``` bash
    ./lab --ex2 --keyLength 192
    ```

    !!! note 
        Output not shown here, it should be similar to the prior two commands.

4. Try with a key length of 384 bits:

    ``` bash
    ./lab --ex2 --keyLength 384
    ```

    ???+ example "Example Output"
        
        ```
        panic: GenerateKey Error: rpc error: code = Unknown desc = CKR_KEY_SIZE_RANGE

        goroutine 1 [running]:
        main.encryptAndDecrypt(0x180, 0xa28e3b, 0x47)
            /home/hyper-protect-lab/go/src/github.com/ibm-developer/ibm-cloud-hyperprotectcrypto/golang/lab/exercise2.go:55 +0x2c89
        main.main()
            /home/hyper-protect-lab/go/src/github.com/ibm-developer/ibm-cloud-hyperprotectcrypto/golang/lab/main.go:73 +0x3a3
        ```

    Don't panic!  (Even if Go did).  I meant for this to happen.

    This exercise uses the *CKM_AES_KEY_GEN* mechanism.  (This is in the first line of the program output).  If we run exercise one again and pipe its output to grep, with a fancy argument to print one extra line, we can see that the minimum key size is 128 bits and the maximum key size is 256 bits.  Try it:

    ``` bash
    ./lab --ex1 | grep CKM_AES_KEY_GEN --after-context 1
    ```

    !!! note 
        The minimum and maximum key sizes are shown in bytes, so you must multiply the values shown by eight to get the sizes in bits.

5. So our key size must be in a range from 128 to 256 bits.  Let's try a key length of 224 bits:

    ``` bash
    ./lab --ex2 --keyLength 224
    ```

    ???+ example "Example Output"

        ```
        panic: GenerateKey Error: rpc error: code = Unknown desc = CKR_KEY_SIZE_RANGE

        goroutine 1 [running]:
        main.encryptAndDecrypt(0xe0, 0xa28e3b, 0x47)
            /home/hyper-protect-lab/go/src/github.com/ibm-developer/ibm-cloud-hyperprotectcrypto/golang/lab/exercise2.go:55 +0x2c89
        main.main()
            /home/hyper-protect-lab/go/src/github.com/ibm-developer/ibm-cloud-hyperprotectcrypto/golang/lab/main.go:73 +0x3a3
        ```

    Is this the new math where 224 is not between 128 and 256? No. What's going on here is that the AES standard supports keys of a certain length-  128, 192, and 256 bits. So while the mechanism information told us the minimum and maximum key size, the expectation is that the programmer knows a little bit about the AES standard when using it!

6. Would you like to encrypt and decrypt your own text?  Try it like this-  feel free to copy and paste what is provided here, or substitute your own witty banter for the value of the `--plaintext` argument.

    ``` bash
    ./lab --ex2 --plaintext "The quick brown fox jumped over the lazy dog"
    ```

    !!! note
        Output not shown, but you should see the message being encrypted and decrypted.  Unless...

    !!! Troubleshooting
        If you received this message in your output, `panic: Failed Encrypt [rpc error: code = Unknown desc = CKR_ARGUMENTS_BAD]`,
        then there is a pretty good chance that you supplied a plaintext value less than twenty-one characters in length.

        This exercise was based on the sample code, which performs the encryption in two steps.
        The first step passed the first twenty bytes of input to the crypto card, and the second step passed the remainder of
        the input. The input was hard-coded and of length greater than twenty bytes, so the sample does not have logic to handle input of twenty bytes or less. The sample has not been modified to handle the case where that second pass isn't needed. 

        Don't worry, in the real world, you can probably solve this problem if you throw enough money at it. :-)


