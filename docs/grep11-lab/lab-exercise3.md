# Exercise 3 - Sign and Verify Data

## Overview of Exercise 3

Exercise three uses *public key cryptography* to demonstrate signing of data and then verifying the signature.

Other terms often used that are associated with public key cryptography are *asymmetric keys* or *public and private key pairs*.

The major characteristic is that an asymmetric key is actually a pair of keys that are related.  One key is called the public key and can be shared with anyone.  The other key is called the private key and is kept secret. 

These key pairs can be used for encryption and decryption, as well as for creating and verifying digital signatures.

## Public key cryptography in simple terms

!!! note "Public key cryptography and encryption"
    When using public key cryptography, the assumption is that anybody in the world could have your public key, but you have kept your private key to yourself.

    For encryption, anybody who has your public key could encrypt data with it and send you that encrypted data, or ciphertext. If an adversary possessed the ciphertext, they could not decrypt it.  Only you, with your private key, can decrypt it.

    Technically, you could encrypt data with your private key that holders of your public key could decrypt, but this use case doesn't make sense- since anybody could have your public key, you have not managed to achieve privacy with your encryption.

!!! note "Public key cryptography and digital signatures"
    Your private key can be used to allow you to digitally "sign" data, which is an attestation that you have seen the data that you have signed, analagous to your signing by pen a letter or contract on a piece of paper. 

    There are different digital signature algorithms, but in general, they follow the following pattern.  Assume you want to digitally sign some data you send to somebody:

    1. You create a digest, or hash, over some data, using some algorithm.  This hash uniquely identifies the data.
    2. You encrypt this hash with your private key. This is your digital signature.
    3. You send the data, and your digital signature, to the intended recipient.
    4. The recipient calculates the digest, or hash, of the data, using the same algorithm that you used in step 1. If the data has remained intact in transit, they will calculate the same hash that you did in step 1.
    5. They then decrypt your signature (from step 2) with your public key. This retrieves in plaintext the hash you signed in step 2.
    6. If the hash they calculated in step 4 matches the hash that you encrypted with your private key in step 2 (and retrieved by them in step 5), then it is proof that you have seen the data they have received.

    The fundamental assumption is that you have kept your private key in your sole possession. If an adversary stole your private key they could impersonate you.

    Also, observe that the validity of this digital signature is independent of whether or not the data that you send is plaintext or ciphertext.

The two most commonly used forms of public key cryptography are *RSA* cryptography and *elliptic curve* cryptography. 

RSA cryptography was developed first and is built upon the difficulty of factoring the product of two very large prime numbers. It is easy for a computer to find two very large prime numbers and multiply them together to obtain a product.  On the other hand, given this product, it is extremely difficult, if not impossible, for even today's powerful computers to figure out what two prime numbers were used to produce that product! It seems like a preposterous statement, but it's true!

Elliptic curve cryptography is based on algebraic equations called elliptic curves. The idea is that a mathematical addition function can be defined on this curve, such that it is very easy for a computer to do a large number of additions on the curve which results in a point on the curve. But if you give someone the equation for the curve, and the point on the curve that you have calculated, but you don't tell them how many additions you did to calculate this point, even today's powerful computers cannot figure out how many additions were performed!  It seems like a preposterous statement, but it's true!

In simple terms, an RSA public key contains the product of the two primes, and the RSA private key contains the hidden information- the two primes. An elliptic curve public key contains all of the information about the curve, including the point that was calculated, except for the number of additions used to calculate the point, while the elliptic curve private key includes the hidden information- the number of additions that were used to create the point.

If I multiplied two small prime numbers like 7 x 13 and gave you the product 91, it probably wouldn't take you too long to factor it back to 7 and 13.  Computers need to use incredibliy large prime numbers in order to calculate a product that another computer can't quickly factor.  That is why RSA keys are usually 2048 or 4096 bits in length.  You can create smaller RSA keys but today's computers can factor the product contained in the public key quicker, the smaller the key.

On the other hand, the size of elliptic curve public keys can be much smaller than RSA public keys, and provide the equivalent strength. That is, a relatively small-sized elliptic curve key is just as difficult for a computer to crack by brute force than it is for a much larger-sized RSA key.  

For this reason, elliptic curve cryptography is becoming increasingly popular- the smaller key size means less computational power is needed to work with these keys, so they are more efficient and use less power, which is more appealing for devices that are typically drawing on battery power, like smartphones and tablets.

EP11 supports both RSA and elliptic curve cryptography, as you saw in the GREP11 sample.  This exercise will dig a little deeper into elliptic curves.

## Run Exercise 3

1.  Run the third exercise:

    ``` bash
    ./lab --ex3
    ```

    ??? example "Example Output"

        ```

        selected curve P521 with ObjectID of 1.3.132.0.35
        Curve ObjectID in ASN.1 BER encoding is [6 5 43 129 4 0 35]

        Generated ECDSA PKCS key pair with mechanism  CKM_EC_KEY_PAIR_GEN

        key pair public key blob length is 286 bytes

        key pair public key blob is [48 129 155 48 16 6 7 42 134 72 206 61 2 1 6 5 43 129 4 0 35 3 129 134 0 4 1 251 93 197 238 11 2 165 14 39 212 27 175 77 65 78 70 252 221 242 195 236 163 3 63 184 70 158 146 51 22 230 181 30 78 101 47 150 133 100 197 101 31 144 59 192 199 237 14 94 86 237 73 52 79 1 45 220 65 57 113 117 113 173 220 70 0 48 137 50 146 141 22 219 122 151 69 232 10 132 29 121 198 108 226 233 9 117 185 16 125 236 113 235 136 54 127 109 229 150 26 117 63 231 242 158 101 248 86 80 17 69 31 217 156 122 140 30 75 132 155 187 35 148 239 49 81 172 10 210 181 169 4 16 3 176 130 49 65 141 152 246 85 132 150 183 226 212 36 40 4 32 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 4 8 255 72 74 131 146 172 234 187 4 8 0 0 0 0 0 0 0 1 4 20 16 1 0 0 0 1 128 128 0 1 128 128 128 1 0 10 0 0 0 1 4 32 217 93 11 148 136 24 62 138 168 213 244 131 187 81 107 102 114 120 167 70 193 179 233 4 32 228 144 160 232 167 125 167]

        key pair private key blob length is 896 bytes

        key pair private key blob is [0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 3 176 130 49 65 141 152 246 85 132 150 183 226 212 36 40 0 0 0 0 0 0 1 38 0 0 0 0 0 0 0 1 18 52 145 242 100 209 252 135 97 118 108 26 68 196 204 214 35 186 208 121 174 78 194 146 179 179 92 26 100 178 30 39 144 58 232 153 47 120 177 131 143 109 255 143 188 55 83 243 168 100 93 113 176 50 33 176 209 178 137 124 13 64 217 65 230 242 127 41 206 53 149 52 238 111 241 174 206 137 141 251 17 145 233 76 219 82 103 108 35 168 57 114 44 179 164 157 12 81 167 193 171 27 42 69 50 108 161 45 227 151 199 15 59 120 177 122 62 66 95 9 115 57 85 29 132 224 146 162 177 224 143 36 125 126 210 113 18 73 145 60 48 109 176 162 41 145 33 242 93 154 122 118 216 252 111 7 253 207 195 227 151 15 112 99 44 177 14 14 193 35 199 207 79 29 219 88 146 132 3 80 128 196 150 37 18 7 192 185 68 38 158 232 84 145 198 24 210 154 34 157 76 53 244 18 31 194 99 80 36 31 225 41 141 69 135 116 43 239 186 193 84 112 103 110 56 148 66 115 76 202 125 45 15 86 124 24 19 247 208 131 51 37 186 172 94 58 155 214 238 131 253 48 86 125 62 240 123 236 237 88 76 218 21 215 202 210 90 29 119 239 2 237 11 242 202 214 116 98 88 90 15 0 12 170 20 96 132 49 224 57 22 113 123 27 19 173 121 50 99 116 175 20 153 192 223 84 121 17 154 88 74 118 245 131 21 179 162 154 44 182 15 66 240 9 154 118 1 122 101 101 103 14 207 47 167 133 152 187 147 221 241 84 223 72 136 243 239 77 65 121 219 208 223 95 175 139 39 100 52 227 185 126 22 143 41 183 28 57 63 238 117 196 172 63 131 116 248 74 165 132 160 77 148 174 44 102 91 29 49 74 230 85 215 83 154 130 46 164 163 212 85 82 103 5 205 233 181 126 160 118 154 65 120 155 89 224 218 132 99 158 134 210 225 219 252 254 167 208 170 164 148 97 226 25 236 152 47 113 74 5 73 177 157 60 120 4 62 87 17 116 190 73 132 124 17 61 46 63 12 184 8 74 177 100 119 151 58 163 126 181 199 95 202 99 112 64 55 97 66 138 83 168 62 160 62 8 160 148 126 215 124 65 228 200 122 192 129 121 18 135 244 95 4 94 0 231 246 49 195 181 35 72 161 110 97 180 135 111 33 28 173 49 235 8 58 56 217 31 159 195 61 195 34 10 160 111 224 53 22 175 9 79 39 142 46 43 91 210 69 183 229 68 114 86 142 221 196 206 234 143 244 49 236 231 253 226 95 36 14 131 74 48 31 144 4 254 235 158 189 98 28 1 100 181 19 139 69 29 166 136 227 85 21 131 130 138 197 249 29 222 65 128 252 16 217 121 157 232 150 126 54 245 116 145 127 175 167 125 46 94 214 38 224 85 172 87 97 163 231 181 218 248 103 211 224 79 122 157 160 76 54 242 13 46 182 10 105 119 6 51 91 11 116 120 2 188 148 108 191 158 100 28 40 158 212 35 94 53 6 59 84 216 207 101 112 40 178 164 252 110 116 39 59 78 78 7 204 241 147 2 234 244 153 214 32 151 187 250 112 162 173 136 214 67 114 195 102 170 240 228 160 123 41 178 7 2 147 66 5 236 100 170 198 233 8 218 163 244 224 211 8 127 36 60 220 185 158 17 171 78 156 168 141 73 86 146 47 252 55 247 12 105 21 170 199 27 69 149 217 78 89 37 125 246 220 29 177 15 18 78 153 170 123 109 248 47 4 130 121 174 88 44 96 61 71 208 112 34 236 111 189 240 195 15 45 176 84 140 35 171 110 124 164 98 0 122 173 69 82 27 118 51 226 30 193 106 51 21 83 194 189 110 18 92 156 198 221 189 211 84 220 207]

        Some of the fields in the private key blob follow: 

        WK ID is [3 176 130 49 65 141 152 246 85 132 150 183 226 212 36 40]
        See 6.7.1 of page 182 of  http://public.dhe.ibm.com/security/cryptocards/pciecc4/EP11/docs/ep11-structure.pdf

        Blob version is [18 52]
        See 3.1.1 on page 141 of http://public.dhe.ibm.com/security/cryptocards/pciecc4/EP11/docs/ep11-structure.pdf

        IV is [145 242 100 209 252 135 97 118 108 26 68 196 204 214]

        Encrypted part is [35 186 208 121 174 78 194 146 179 179 92 26 100 178 30 39 144 58 232 153 47 120 177 131 143 109 255 143 188 55 83 243 168 100 93 113 176 50 33 176 209 178 137 124 13 64 217 65 230 242 127 41 206 53 149 52 238 111 241 174 206 137 141 251 17 145 233 76 219 82 103 108 35 168 57 114 44 179 164 157 12 81 167 193 171 27 42 69 50 108 161 45 227 151 199 15 59 120 177 122 62 66 95 9 115 57 85 29 132 224 146 162 177 224 143 36 125 126 210 113 18 73 145 60 48 109 176 162 41 145 33 242 93 154 122 118 216 252 111 7 253 207 195 227 151 15 112 99 44 177 14 14 193 35 199 207 79 29 219 88 146 132 3 80 128 196 150 37 18 7 192 185 68 38 158 232 84 145 198 24 210 154 34 157 76 53 244 18 31 194 99 80 36 31 225 41 141 69 135 116 43 239 186 193 84 112 103 110 56 148 66 115 76 202 125 45 15 86 124 24 19 247 208 131 51 37 186 172 94 58 155 214 238 131 253 48 86 125 62 240 123 236 237 88 76 218 21 215 202 210 90 29 119 239 2 237 11 242 202 214 116 98 88 90 15 0 12 170 20 96 132 49 224 57 22 113 123 27 19 173 121 50 99 116 175 20 153 192 223 84 121 17 154 88 74 118 245 131 21 179 162 154 44 182 15 66 240 9 154 118 1 122 101 101 103 14 207 47 167 133 152 187 147 221 241 84 223 72 136 243 239 77 65 121 219 208 223 95 175 139 39 100 52 227 185 126 22 143 41 183 28 57 63 238 117 196 172 63 131 116 248 74 165 132 160 77 148 174 44 102 91 29 49 74 230 85 215 83 154 130 46 164 163 212 85 82 103 5 205 233 181 126 160 118 154 65 120 155 89 224 218 132 99 158 134 210 225 219 252 254 167 208 170 164 148 97 226 25 236 152 47 113 74 5 73 177 157 60 120 4 62 87 17 116 190 73 132 124 17 61 46 63 12 184 8 74 177 100 119 151 58 163 126 181 199 95 202 99 112 64 55 97 66 138 83 168 62 160 62 8 160 148 126 215 124 65 228 200 122 192 129 121 18 135 244 95 4 94 0 231 246 49 195 181 35 72 161 110 97 180 135 111 33 28 173 49 235 8 58 56 217 31 159 195 61 195 34 10 160 111 224 53 22 175 9 79 39 142 46 43 91 210 69 183 229 68 114 86 142 221 196 206 234 143 244 49 236 231 253 226 95 36 14 131 74 48 31 144 4 254 235 158 189 98 28 1 100 181 19 139 69 29 166 136 227 85 21 131 130 138 197 249 29 222 65 128 252 16 217 121 157 232 150 126 54 245 116 145 127 175 167 125 46 94 214 38 224 85 172 87 97 163 231 181 218 248 103 211 224 79 122 157 160 76 54 242 13 46 182 10 105 119 6 51 91 11 116 120 2 188 148 108 191 158 100 28 40 158 212 35 94 53 6 59 84 216 207 101 112 40 178 164 252 110 116 39 59 78 78 7 204 241 147 2 234 244 153 214 32 151 187 250 112 162 173 136 214 67 114 195 102 170 240 228 160 123 41 178 7 2 147 66 5 236 100 170 198 233 8 218 163 244 224 211 8 127 36 60 220 185 158 17 171 78 156 168 141 73 86 146 47 252 55 247 12 105 21 170 199 27 69 149 217 78 89 37 125 246 220 29 177 15 18 78 153 170 123 109 248 47 4 130 121 174 88 44 96 61 71 208 112 34 236 111 189 240 195 15 45 176 84 140 35 171]

        MAC is [110 124 164 98 0 122 173 69 82 27 118 51 226 30 193 106 51 21 83 194 189 110 18 92 156 198 221 189 211 84 220 207]
        MAC is 32 bytes, see field 15 in 3.1 on page 140 of http://public.dhe.ibm.com/security/cryptocards/pciecc4/EP11/docs/ep11-structure.pdf

        -----BEGIN HSM ENCRYPTED PRIVATE KEY-----
        AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAADsIIxQY2Y9lWElrfi1CQo
        AAAAAAAAASYAAAAAAAAAARI0kfJk0fyHYXZsGkTEzNYjutB5rk7CkrOzXBpksh4n
        kDromS94sYOPbf+PvDdT86hkXXGwMiGw0bKJfA1A2UHm8n8pzjWVNO5v8a7OiY37
        EZHpTNtSZ2wjqDlyLLOknQxRp8GrGypFMmyhLeOXxw87eLF6PkJfCXM5VR2E4JKi
        seCPJH1+0nESSZE8MG2woimRIfJdmnp22PxvB/3Pw+OXD3BjLLEODsEjx89PHdtY
        koQDUIDEliUSB8C5RCae6FSRxhjSmiKdTDX0Eh/CY1AkH+EpjUWHdCvvusFUcGdu
        OJRCc0zKfS0PVnwYE/fQgzMluqxeOpvW7oP9MFZ9PvB77O1YTNoV18rSWh137wLt
        C/LK1nRiWFoPAAyqFGCEMeA5FnF7GxOteTJjdK8UmcDfVHkRmlhKdvWDFbOimiy2
        D0LwCZp2AXplZWcOzy+nhZi7k93xVN9IiPPvTUF529DfX6+LJ2Q047l+Fo8ptxw5
        P+51xKw/g3T4SqWEoE2UrixmWx0xSuZV11Oagi6ko9RVUmcFzem1fqB2mkF4m1ng
        2oRjnobS4dv8/qfQqqSUYeIZ7JgvcUoFSbGdPHgEPlcRdL5JhHwRPS4/DLgISrFk
        d5c6o361x1/KY3BAN2FCilOoPqA+CKCUftd8QeTIesCBeRKH9F8EXgDn9jHDtSNI
        oW5htIdvIRytMesIOjjZH5/DPcMiCqBv4DUWrwlPJ44uK1vSRbflRHJWjt3EzuqP
        9DHs5/3iXyQOg0owH5AE/uuevWIcAWS1E4tFHaaI41UVg4KKxfkd3kGA/BDZeZ3o
        ln429XSRf6+nfS5e1ibgVaxXYaPntdr4Z9PgT3qdoEw28g0utgppdwYzWwt0eAK8
        lGy/nmQcKJ7UI141BjtU2M9lcCiypPxudCc7Tk4HzPGTAur0mdYgl7v6cKKtiNZD
        csNmqvDkoHspsgcCk0IF7GSqxukI2qP04NMIfyQ83LmeEatOnKiNSVaSL/w39wxp
        FarHG0WV2U5ZJX323B2xDxJOmap7bfgvBIJ5rlgsYD1H0HAi7G+98MMPLbBUjCOr
        bnykYgB6rUVSG3Yz4h7BajMVU8K9bhJcnMbdvdNU3M8=
        -----END HSM ENCRYPTED PRIVATE KEY-----

        -----BEGIN PUBLIC KEY-----
        MIGbMBAGByqGSM49AgEGBSuBBAAjA4GGAAQB+13F7gsCpQ4n1BuvTUFORvzd8sPs
        owM/uEaekjMW5rUeTmUvloVkxWUfkDvAx+0OXlbtSTRPAS3cQTlxdXGt3EYAMIky
        ko0W23qXRegKhB15xmzi6Ql1uRB97HHriDZ/beWWGnU/5/KeZfhWUBFFH9mceowe
        S4SbuyOU7zFRrArStakEEAOwgjFBjZj2VYSWt+LUJCgEIAAAAAAAAAAAAAAAAAAA
        AAAAAAAAAAAAAAAAAAAAAAAABAj/SEqDkqzquwQIAAAAAAAAAAEEFBABAAAAAYCA
        AAGAgIABAAoAAAABBCDZXQuUiBg+iqjV9IO7UWtmcninRsGz6QQg5JCg6Kd9pw==
        -----END PUBLIC KEY-----

        Data signed
        Signature verified

        ```

    The first line of the output shows the Object ID of the curve that was used. These Object IDs are assigned by standards bodies. The Object ID used here is `1.3.132.0.35`.  Stick 1.3.132.0.35 in your favorite Google machine and see what you find. Or take my word for it that this identifies the *secp521r1* curve.

    The second line of the output shows ASN.1 BER encoding of 1.3.132.0.35, which is *6 5 43 129 4 0 35*.  If you were to look up how ASN.1 BER encoding works, your first question would be, "Why?".  Don't go there now.  But I bring it up because you will notice that a few lines down in the output, in the *key pair public key blob* section, you'll see that encoded string, *6 5 43 129 4 0 35*, about 14 bytes from the beginning.  In fact, while the whole point of using the GREP11 server and the Crypto Express 7S cards is to keep secrets from an adversary, the information shown in the *key pair public key blob* section is "in the clear"- you could give this to your adversary without fear of having your secrets compromised.

    The list of decimal bytes of these keys is mostly gibberish to us humans, I'm not sure why I even bothered to print them out, but take a look at the output at the end, where there are several lines between a header and footer for HSM ENCRYPTED PRIVATE KEY (BEGIN and END), and several lines between a header and footer for a PUBLIC KEY.

    To use these keys in the future, you would need to persist both of these to separate files. You wouldn't have a reason to give your adversary the HSM ENCRYPTED PRIVATE KEY, but, if he did obtain it, it would be of no value to him unless he was able to access your Crypto Express 7S card that had the same root wrapping key. The contents of this file are encrypted and are usuable only within the Crypto Express card that created it, or in another Crypto Express card that had the same root wrapping key.

    Since the PUBLIC KEY is, by definition, shareable with the world, we can use the common `openssl` utility to look at it. 

    Try it like this:

    1. Copy the lines from near the bottom of your output that have your public key , including the *BEGIN PUBLIC KEY* and *END PUBLIC KEY* header and footer lines, and the dashes, into your clipboard

    2. In a terminal window,  type `echo "` (that is a single pair of double-quotes) and then paste your clipboard contents onto the command line

    3.  Type `" | openssl ec -in - -pubin -noout -text` and then press Enter

    ??? example "You should see output similar to this:"

        ```
        read EC key
        Public-Key: (521 bit)
        pub:
            04:00:59:fe:0c:b3:d0:6e:b2:d7:b0:c0:05:86:66:
            dd:62:bc:fe:96:5c:fa:08:6d:f3:be:5f:6d:a1:3d:
            8f:40:13:83:23:ec:34:e4:e5:20:6c:f9:75:34:bd:
            30:cd:be:1f:5b:aa:ee:44:99:d9:ae:69:50:24:c7:
            82:7c:f4:f2:18:d1:b0:00:be:5c:64:f5:dd:16:79:
            eb:d4:9c:28:ae:43:9b:f4:13:a4:f3:db:3c:a2:e5:
            52:bb:0f:9f:e1:c4:a4:c1:29:63:4e:bc:2f:73:4c:
            da:fd:4f:42:63:0d:3d:9d:bb:29:76:57:85:13:57:
            3a:ae:8f:20:64:0a:cf:c2:f1:41:d3:fc:8a
        ASN1 OID: secp521r1
        NIST CURVE: P-521
        ```

    !!! tip
        If you are a Linux guru you could create a file with *vi* and copy the key into the file, and then provide that as input to the openssl command, e.g., `openssl ec -in baz -pubin -noout -text` if *baz* was the file you created.

    In the exercise output I first printed the public key as an array of decimal numbers (worthless!),  then I printed it in PEM-encoded format (boring!) but now you see the key in hexadecimal format (that's incredible!!). You also get the added bonus of knowing it's a legit public key since the `openssl` utility made sense of it.

2. Try  using a different curve:

    ``` bash
    ./lab --ex3 --curve P224
    ```

    ??? example "Example Output"

        ```

        selected curve P224 with ObjectID of 1.3.132.0.33
        Curve ObjectID in ASN.1 BER encoding is [6 5 43 129 4 0 33]

        Generated ECDSA PKCS key pair with mechanism  CKM_EC_KEY_PAIR_GEN

        key pair public key blob length is 208 bytes

        key pair public key blob is [48 78 48 16 6 7 42 134 72 206 61 2 1 6 5 43 129 4 0 33 3 58 0 4 177 164 170 83 241 4 140 49 236 115 12 22 130 206 48 90 174 181 135 16 59 31 11 122 14 254 13 172 129 64 249 1 40 44 231 155 52 110 38 67 175 244 45 81 157 5 213 125 224 199 126 65 251 200 159 53 4 16 3 176 130 49 65 141 152 246 85 132 150 183 226 212 36 40 4 32 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 4 8 6 63 231 63 240 230 179 88 4 8 0 0 0 0 0 0 0 1 4 20 16 1 0 0 0 1 128 128 0 1 128 128 128 1 0 10 0 0 0 1 4 32 90 61 58 127 30 244 119 154 65 27 170 14 182 232 45 237 92 61 188 238 138 237 84 7 5 209 228 172 195 235 219 189]

        key pair private key blob length is 544 bytes

        key pair private key blob is [0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 3 176 130 49 65 141 152 246 85 132 150 183 226 212 36 40 0 0 0 0 0 0 1 38 0 0 0 0 0 0 0 1 18 52 1 7 210 186 20 15 180 182 68 217 17 251 23 142 218 85 206 228 75 16 37 108 9 47 216 5 160 29 100 235 78 164 63 25 232 181 251 214 113 69 26 198 9 222 119 149 66 64 233 183 172 93 96 41 223 99 201 110 138 227 51 26 197 166 9 8 190 40 129 254 49 89 54 38 209 21 60 137 118 113 174 145 254 145 152 119 73 114 11 144 41 25 106 152 140 167 192 88 82 199 100 234 18 255 129 213 111 25 126 2 200 3 44 90 252 84 48 191 106 98 2 77 78 7 165 189 227 52 167 78 188 5 185 254 178 206 180 171 47 110 54 38 87 16 191 167 134 237 48 49 153 168 54 225 251 75 223 3 55 239 48 119 76 89 103 20 30 222 104 167 219 210 46 17 75 224 196 191 55 238 236 156 205 119 182 199 12 58 144 248 124 216 181 38 11 17 139 144 123 31 108 76 171 91 154 112 6 160 188 2 181 110 212 46 66 153 211 21 184 1 51 194 12 245 73 155 192 206 80 213 51 187 166 3 207 127 151 127 59 158 192 7 92 113 247 118 108 55 217 84 183 34 24 144 57 215 193 96 179 93 2 190 84 85 119 199 179 96 119 2 9 160 231 251 214 187 5 206 224 207 111 52 58 181 131 237 11 245 106 181 223 217 240 98 162 190 11 116 18 46 225 218 17 130 79 19 4 171 94 129 197 59 201 54 63 31 60 90 84 225 223 121 94 173 42 103 3 204 128 187 143 209 78 80 151 157 90 44 76 20 254 18 96 197 70 36 135 24 13 156 77 112 196 22 35 123 199 169 17 195 156 0 212 50 140 50 126 70 132 126 149 172 94 173 49 156 173 207 208 109 166 88 148 204 25 122 119 58 130 215 130 101 13 235 96 29 210 130 229 51 232 250 206 124 162 68 112 3 191 129 184 195 34 186 72 103 159 228 155 244 43 108 205 37 82 176 243 200 233 88 91 31 243 33 232 44 178 19 57 182 239 63 238 130 33 1 127 250 215 109 121 187 134 218 173 74 112 88 230 152 90 147 20 43 28 254 173 28 25 255 0 147 78 51 231 175 190 29]

        Some of the fields in the private key blob follow: 

        WK ID is [3 176 130 49 65 141 152 246 85 132 150 183 226 212 36 40]
        See 6.7.1 of page 182 of  http://public.dhe.ibm.com/security/cryptocards/pciecc4/EP11/docs/ep11-structure.pdf

        Blob version is [18 52]
        See 3.1.1 on page 141 of http://public.dhe.ibm.com/security/cryptocards/pciecc4/EP11/docs/ep11-structure.pdf

        IV is [1 7 210 186 20 15 180 182 68 217 17 251 23 142]

        Encrypted part is [218 85 206 228 75 16 37 108 9 47 216 5 160 29 100 235 78 164 63 25 232 181 251 214 113 69 26 198 9 222 119 149 66 64 233 183 172 93 96 41 223 99 201 110 138 227 51 26 197 166 9 8 190 40 129 254 49 89 54 38 209 21 60 137 118 113 174 145 254 145 152 119 73 114 11 144 41 25 106 152 140 167 192 88 82 199 100 234 18 255 129 213 111 25 126 2 200 3 44 90 252 84 48 191 106 98 2 77 78 7 165 189 227 52 167 78 188 5 185 254 178 206 180 171 47 110 54 38 87 16 191 167 134 237 48 49 153 168 54 225 251 75 223 3 55 239 48 119 76 89 103 20 30 222 104 167 219 210 46 17 75 224 196 191 55 238 236 156 205 119 182 199 12 58 144 248 124 216 181 38 11 17 139 144 123 31 108 76 171 91 154 112 6 160 188 2 181 110 212 46 66 153 211 21 184 1 51 194 12 245 73 155 192 206 80 213 51 187 166 3 207 127 151 127 59 158 192 7 92 113 247 118 108 55 217 84 183 34 24 144 57 215 193 96 179 93 2 190 84 85 119 199 179 96 119 2 9 160 231 251 214 187 5 206 224 207 111 52 58 181 131 237 11 245 106 181 223 217 240 98 162 190 11 116 18 46 225 218 17 130 79 19 4 171 94 129 197 59 201 54 63 31 60 90 84 225 223 121 94 173 42 103 3 204 128 187 143 209 78 80 151 157 90 44 76 20 254 18 96 197 70 36 135 24 13 156 77 112 196 22 35 123 199 169 17 195 156 0 212 50 140 50 126 70 132 126 149 172 94 173 49 156 173 207 208 109 166 88 148 204 25 122 119 58 130 215 130 101 13 235 96 29 210 130 229 51 232 250 206 124 162 68 112 3 191 129 184 195 34 186 72 103 159 228 155 244 43 108 205 37 82 176 243 200 233 88 91 31 243 33 232 44 178 19 57 182 239 63 238 130 33 1]

        MAC is [127 250 215 109 121 187 134 218 173 74 112 88 230 152 90 147 20 43 28 254 173 28 25 255 0 147 78 51 231 175 190 29]
        MAC is 32 bytes, see field 15 in 3.1 on page 140 of http://public.dhe.ibm.com/security/cryptocards/pciecc4/EP11/docs/ep11-structure.pdf

        -----BEGIN HSM ENCRYPTED PRIVATE KEY-----
        AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAADsIIxQY2Y9lWElrfi1CQo
        AAAAAAAAASYAAAAAAAAAARI0AQfSuhQPtLZE2RH7F47aVc7kSxAlbAkv2AWgHWTr
        TqQ/Gei1+9ZxRRrGCd53lUJA6besXWAp32PJborjMxrFpgkIviiB/jFZNibRFTyJ
        dnGukf6RmHdJcguQKRlqmIynwFhSx2TqEv+B1W8ZfgLIAyxa/FQwv2piAk1OB6W9
        4zSnTrwFuf6yzrSrL242JlcQv6eG7TAxmag24ftL3wM37zB3TFlnFB7eaKfb0i4R
        S+DEvzfu7JzNd7bHDDqQ+HzYtSYLEYuQex9sTKtbmnAGoLwCtW7ULkKZ0xW4ATPC
        DPVJm8DOUNUzu6YDz3+XfzuewAdccfd2bDfZVLciGJA518Fgs10CvlRVd8ezYHcC
        CaDn+9a7Bc7gz280OrWD7Qv1arXf2fBior4LdBIu4doRgk8TBKtegcU7yTY/Hzxa
        VOHfeV6tKmcDzIC7j9FOUJedWixMFP4SYMVGJIcYDZxNcMQWI3vHqRHDnADUMowy
        fkaEfpWsXq0xnK3P0G2mWJTMGXp3OoLXgmUN62Ad0oLlM+j6znyiRHADv4G4wyK6
        SGef5Jv0K2zNJVKw88jpWFsf8yHoLLITObbvP+6CIQF/+tdtebuG2q1KcFjmmFqT
        FCsc/q0cGf8Ak04z56++HQ==
        -----END HSM ENCRYPTED PRIVATE KEY-----

        -----BEGIN PUBLIC KEY-----
        ME4wEAYHKoZIzj0CAQYFK4EEACEDOgAEsaSqU/EEjDHscwwWgs4wWq61hxA7Hwt6
        Dv4NrIFA+QEoLOebNG4mQ6/0LVGdBdV94Md+QfvInzUEEAOwgjFBjZj2VYSWt+LU
        JCgEIAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAABAgGP+c/8OazWAQI
        AAAAAAAAAAEEFBABAAAAAYCAAAGAgIABAAoAAAABBCBaPTp/HvR3mkEbqg626C3t
        XD287ortVAcF0eSsw+vbvQ==
        -----END PUBLIC KEY-----

        Data signed
        Signature verified

        ```

3. Try the P256 curve:

    ```
    ./lab --ex3 --curve P256
    ```

    ??? example "Example output"

        ```

        selected curve P256 with ObjectID of 1.2.840.10045.3.1.7
        Curve ObjectID in ASN.1 BER encoding is [6 8 42 134 72 206 61 3 1 7]

        Generated ECDSA PKCS key pair with mechanism  CKM_EC_KEY_PAIR_GEN

        key pair public key blob length is 219 bytes

        key pair public key blob is [48 89 48 19 6 7 42 134 72 206 61 2 1 6 8 42 134 72 206 61 3 1 7 3 66 0 4 110 106 106 221 176 75 119 181 154 179 115 108 192 148 127 80 203 20 154 165 194 20 40 117 94 117 223 125 118 133 226 230 18 128 20 46 239 61 241 136 41 9 78 202 124 188 192 93 181 229 245 210 78 187 156 152 192 132 255 162 17 51 187 117 4 16 3 176 130 49 65 141 152 246 85 132 150 183 226 212 36 40 4 32 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 4 8 203 203 135 115 66 126 231 150 4 8 0 0 0 0 0 0 0 1 4 20 16 1 0 0 0 1 128 128 0 1 128 128 128 1 0 10 0 0 0 1 4 32 117 106 180 43 131 151 71 204 222 211 38 32 214 138 208 98 235 7 172 87 236 205 51 122 54 65 64 58 70 89 127 218]

        key pair private key blob length is 576 bytes

        key pair private key blob is [0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 3 176 130 49 65 141 152 246 85 132 150 183 226 212 36 40 0 0 0 0 0 0 1 38 0 0 0 0 0 0 0 1 18 52 234 153 96 242 234 92 129 26 151 136 72 248 23 66 196 52 4 170 78 111 124 89 151 169 22 232 12 154 65 246 71 238 229 101 98 135 230 50 24 66 130 82 190 65 48 151 138 8 183 43 229 56 95 235 90 163 68 81 9 77 56 28 104 192 150 105 31 193 133 24 224 125 104 86 165 104 162 87 247 209 66 173 168 45 232 4 229 134 211 133 24 229 196 10 32 90 129 146 138 115 39 163 206 102 141 49 142 143 55 114 160 210 56 132 237 137 157 136 203 0 20 217 144 233 140 198 26 179 60 218 95 201 29 214 100 144 55 226 195 23 83 105 50 175 174 214 216 130 2 71 17 191 197 26 180 89 147 168 61 236 49 230 64 190 217 199 13 244 173 182 255 91 144 51 231 177 252 232 210 156 130 70 14 30 129 255 82 247 162 116 213 100 248 100 190 125 48 3 168 85 249 192 75 26 221 95 113 27 1 123 130 248 176 164 140 61 141 190 157 241 193 100 2 180 163 73 141 252 213 35 213 158 95 46 85 0 94 159 101 16 31 97 109 63 64 35 238 255 208 175 121 180 84 162 66 210 119 10 131 119 187 27 116 152 2 41 62 175 176 244 123 70 42 51 244 113 238 102 113 195 54 242 80 148 90 145 226 10 180 122 94 143 119 125 187 118 7 28 184 214 205 205 97 18 71 235 186 245 199 246 57 3 208 121 40 73 6 28 64 217 154 216 108 59 46 158 80 133 8 196 11 223 201 199 234 77 99 219 238 244 6 254 96 55 5 88 109 181 57 85 82 64 67 46 13 114 56 74 117 215 61 161 26 37 221 210 18 229 131 11 232 38 101 190 54 241 253 217 100 80 205 45 113 51 237 105 252 159 11 126 74 152 22 138 45 30 188 157 151 101 139 134 16 221 153 56 216 167 210 214 79 150 96 170 83 161 118 219 162 45 113 179 106 87 147 99 222 42 122 194 24 249 53 210 222 206 2 74 232 110 135 6 12 110 53 2 15 144 180 185 58 249 41 54 36 128 61 116 221 158 154 95 142 32 213 89 48 209 47 155 239 2 142 171 218 207 130 140 18 125 86 224 20 5 4 83 108 163 82 243 62 254 77 103 87 90 32 112 50 169 54 139 200 219 92 84 191 177 139 94]

        Some of the fields in the private key blob follow: 

        WK ID is [3 176 130 49 65 141 152 246 85 132 150 183 226 212 36 40]
        See 6.7.1 of page 182 of  http://public.dhe.ibm.com/security/cryptocards/pciecc4/EP11/docs/ep11-structure.pdf

        Blob version is [18 52]
        See 3.1.1 on page 141 of http://public.dhe.ibm.com/security/cryptocards/pciecc4/EP11/docs/ep11-structure.pdf

        IV is [234 153 96 242 234 92 129 26 151 136 72 248 23 66]

        Encrypted part is [196 52 4 170 78 111 124 89 151 169 22 232 12 154 65 246 71 238 229 101 98 135 230 50 24 66 130 82 190 65 48 151 138 8 183 43 229 56 95 235 90 163 68 81 9 77 56 28 104 192 150 105 31 193 133 24 224 125 104 86 165 104 162 87 247 209 66 173 168 45 232 4 229 134 211 133 24 229 196 10 32 90 129 146 138 115 39 163 206 102 141 49 142 143 55 114 160 210 56 132 237 137 157 136 203 0 20 217 144 233 140 198 26 179 60 218 95 201 29 214 100 144 55 226 195 23 83 105 50 175 174 214 216 130 2 71 17 191 197 26 180 89 147 168 61 236 49 230 64 190 217 199 13 244 173 182 255 91 144 51 231 177 252 232 210 156 130 70 14 30 129 255 82 247 162 116 213 100 248 100 190 125 48 3 168 85 249 192 75 26 221 95 113 27 1 123 130 248 176 164 140 61 141 190 157 241 193 100 2 180 163 73 141 252 213 35 213 158 95 46 85 0 94 159 101 16 31 97 109 63 64 35 238 255 208 175 121 180 84 162 66 210 119 10 131 119 187 27 116 152 2 41 62 175 176 244 123 70 42 51 244 113 238 102 113 195 54 242 80 148 90 145 226 10 180 122 94 143 119 125 187 118 7 28 184 214 205 205 97 18 71 235 186 245 199 246 57 3 208 121 40 73 6 28 64 217 154 216 108 59 46 158 80 133 8 196 11 223 201 199 234 77 99 219 238 244 6 254 96 55 5 88 109 181 57 85 82 64 67 46 13 114 56 74 117 215 61 161 26 37 221 210 18 229 131 11 232 38 101 190 54 241 253 217 100 80 205 45 113 51 237 105 252 159 11 126 74 152 22 138 45 30 188 157 151 101 139 134 16 221 153 56 216 167 210 214 79 150 96 170 83 161 118 219 162 45 113 179 106 87 147 99 222 42 122 194 24 249 53 210 222 206 2 74 232 110 135 6 12 110 53 2 15 144 180 185 58 249 41 54 36 128 61 116 221 158 154 95 142 32 213 89 48 209 47 155 239 2 142 171 218 207 130 140]

        MAC is [18 125 86 224 20 5 4 83 108 163 82 243 62 254 77 103 87 90 32 112 50 169 54 139 200 219 92 84 191 177 139 94]
        MAC is 32 bytes, see field 15 in 3.1 on page 140 of http://public.dhe.ibm.com/security/cryptocards/pciecc4/EP11/docs/ep11-structure.pdf

        -----BEGIN HSM ENCRYPTED PRIVATE KEY-----
        AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAADsIIxQY2Y9lWElrfi1CQo
        AAAAAAAAASYAAAAAAAAAARI06plg8upcgRqXiEj4F0LENASqTm98WZepFugMmkH2
        R+7lZWKH5jIYQoJSvkEwl4oItyvlOF/rWqNEUQlNOBxowJZpH8GFGOB9aFalaKJX
        99FCragt6ATlhtOFGOXECiBagZKKcyejzmaNMY6PN3Kg0jiE7YmdiMsAFNmQ6YzG
        GrM82l/JHdZkkDfiwxdTaTKvrtbYggJHEb/FGrRZk6g97DHmQL7Zxw30rbb/W5Az
        57H86NKcgkYOHoH/UveidNVk+GS+fTADqFX5wEsa3V9xGwF7gviwpIw9jb6d8cFk
        ArSjSY381SPVnl8uVQBen2UQH2FtP0Aj7v/Qr3m0VKJC0ncKg3e7G3SYAik+r7D0
        e0YqM/Rx7mZxwzbyUJRakeIKtHpej3d9u3YHHLjWzc1hEkfruvXH9jkD0HkoSQYc
        QNma2Gw7Lp5QhQjEC9/Jx+pNY9vu9Ab+YDcFWG21OVVSQEMuDXI4SnXXPaEaJd3S
        EuWDC+gmZb428f3ZZFDNLXEz7Wn8nwt+SpgWii0evJ2XZYuGEN2ZONin0tZPlmCq
        U6F226ItcbNqV5Nj3ip6whj5NdLezgJK6G6HBgxuNQIPkLS5OvkpNiSAPXTdnppf
        jiDVWTDRL5vvAo6r2s+CjBJ9VuAUBQRTbKNS8z7+TWdXWiBwMqk2i8jbXFS/sYte
        -----END HSM ENCRYPTED PRIVATE KEY-----

        -----BEGIN PUBLIC KEY-----
        MFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAEbmpq3bBLd7Was3NswJR/UMsUmqXC
        FCh1XnXffXaF4uYSgBQu7z3xiCkJTsp8vMBdteX10k67nJjAhP+iETO7dQQQA7CC
        MUGNmPZVhJa34tQkKAQgAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAE
        CMvLh3NCfueWBAgAAAAAAAAAAQQUEAEAAAABgIAAAYCAgAEACgAAAAEEIHVqtCuD
        l0fM3tMmINaK0GLrB6xX7M0zejZBQDpGWX/a
        -----END PUBLIC KEY-----

        Data signed
        Signature verified

        ```

4. Finally, try the fourth suupported curve, P384:

    ```
    ./lab --ex3 --curve P384
    ```

    ??? example "Example output"

        ```
 
        selected curve P384 with ObjectID of 1.3.132.0.34
        Curve ObjectID in ASN.1 BER encoding is [6 5 43 129 4 0 34]

        Generated ECDSA PKCS key pair with mechanism  CKM_EC_KEY_PAIR_GEN

        key pair public key blob length is 248 bytes

        key pair public key blob is [48 118 48 16 6 7 42 134 72 206 61 2 1 6 5 43 129 4 0 34 3 98 0 4 20 236 106 175 76 209 142 54 136 119 12 96 68 207 237 15 207 93 133 101 224 190 191 207 252 8 13 111 41 146 195 110 245 102 246 21 126 198 152 204 142 142 123 85 205 73 71 95 199 156 111 125 60 137 24 103 37 211 215 222 89 119 181 242 0 63 164 49 14 7 40 199 86 111 168 136 8 249 164 117 253 72 33 144 66 15 126 131 230 42 32 37 210 78 27 99 4 16 3 176 130 49 65 141 152 246 85 132 150 183 226 212 36 40 4 32 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 4 8 31 146 195 59 47 0 221 196 4 8 0 0 0 0 0 0 0 1 4 20 16 1 0 0 0 1 128 128 0 1 128 128 128 1 0 10 0 0 0 1 4 32 166 134 66 17 102 8 140 118 31 64 4 242 234 134 34 72 132 116 245 10 224 174 74 101 243 172 223 184 156 51 28 103]

        key pair private key blob length is 720 bytes

        key pair private key blob is [0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 3 176 130 49 65 141 152 246 85 132 150 183 226 212 36 40 0 0 0 0 0 0 1 38 0 0 0 0 0 0 0 1 18 52 153 221 86 12 9 88 111 233 79 50 0 98 38 116 119 42 179 162 115 61 69 173 2 50 40 5 241 34 125 95 215 44 94 60 200 193 50 47 70 234 63 92 12 149 107 163 199 247 244 21 145 94 176 100 72 240 241 146 134 62 70 238 188 1 31 91 136 108 73 110 218 192 207 194 239 183 109 245 140 135 33 148 169 75 112 120 9 173 218 117 181 74 141 98 52 108 110 115 135 113 187 153 210 17 27 101 149 108 233 171 190 217 178 69 154 188 13 79 167 107 197 222 34 246 90 23 80 87 71 32 39 100 100 38 41 239 210 168 65 58 118 220 16 7 32 148 54 163 80 1 199 139 165 132 161 234 172 7 87 11 31 106 203 50 250 123 9 145 5 40 155 103 123 16 113 60 61 210 215 238 164 150 191 97 200 231 170 17 180 184 90 104 114 10 114 61 129 222 6 139 56 248 177 110 10 53 234 143 90 10 149 134 174 170 32 237 224 29 124 102 218 206 171 105 10 18 16 233 191 119 26 12 167 215 135 23 95 83 89 171 154 14 249 247 35 202 88 83 220 5 191 52 71 15 64 20 21 62 55 241 158 98 38 95 137 193 201 253 122 71 25 10 128 22 53 60 199 175 192 194 104 68 187 83 136 228 246 102 194 153 119 139 178 33 170 80 237 242 75 179 107 184 194 221 45 216 161 217 150 100 224 22 7 94 142 242 114 3 52 237 105 123 39 61 185 226 37 38 159 64 208 219 25 170 246 223 126 122 120 135 86 193 101 6 105 148 156 12 77 162 221 200 34 202 29 228 198 5 193 112 96 95 16 225 1 6 41 252 234 218 227 64 95 16 221 124 2 4 42 209 138 212 11 35 149 12 240 152 187 36 87 60 156 4 138 76 214 193 233 17 174 49 119 133 96 215 49 137 139 45 36 162 230 79 83 100 87 83 13 159 146 143 170 144 34 248 76 97 244 226 63 124 122 121 35 177 81 219 119 21 42 42 40 92 37 220 93 118 245 215 76 129 128 251 187 202 225 112 82 171 55 8 158 70 93 209 70 31 31 251 22 170 11 215 35 190 122 0 48 146 182 84 180 106 192 119 0 227 114 134 98 247 100 13 30 56 20 232 87 56 120 132 197 194 244 138 6 31 224 220 50 114 202 129 226 170 8 168 37 184 206 16 11 55 229 59 26 38 6 74 4 191 232 77 70 242 232 80 3 144 192 248 102 68 155 22 34 97 12 221 63 106 55 248 185 169 207 239 178 95 71 220 127 87 74 14 241 233 45 248 208 51 92 103 59 39 218 52 53 177 164 7 135 195 99 49 135 89 18 132 8 185 51 41 0 0 210 46 166 22 59 21 223 44 65 60 83 41 211 102 10 123 89 57 98 87 183 212 197 96 149 236 104 175 109 207 180 77 52 123 170 159 77 29 255 146 246 94 95 71 63 156 183 227 45 82 160 148 152 87 53 42 161 58]

        Some of the fields in the private key blob follow: 

        WK ID is [3 176 130 49 65 141 152 246 85 132 150 183 226 212 36 40]
        See 6.7.1 of page 182 of  http://public.dhe.ibm.com/security/cryptocards/pciecc4/EP11/docs/ep11-structure.pdf

        Blob version is [18 52]
        See 3.1.1 on page 141 of http://public.dhe.ibm.com/security/cryptocards/pciecc4/EP11/docs/ep11-structure.pdf

        IV is [153 221 86 12 9 88 111 233 79 50 0 98 38 116]

        Encrypted part is [119 42 179 162 115 61 69 173 2 50 40 5 241 34 125 95 215 44 94 60 200 193 50 47 70 234 63 92 12 149 107 163 199 247 244 21 145 94 176 100 72 240 241 146 134 62 70 238 188 1 31 91 136 108 73 110 218 192 207 194 239 183 109 245 140 135 33 148 169 75 112 120 9 173 218 117 181 74 141 98 52 108 110 115 135 113 187 153 210 17 27 101 149 108 233 171 190 217 178 69 154 188 13 79 167 107 197 222 34 246 90 23 80 87 71 32 39 100 100 38 41 239 210 168 65 58 118 220 16 7 32 148 54 163 80 1 199 139 165 132 161 234 172 7 87 11 31 106 203 50 250 123 9 145 5 40 155 103 123 16 113 60 61 210 215 238 164 150 191 97 200 231 170 17 180 184 90 104 114 10 114 61 129 222 6 139 56 248 177 110 10 53 234 143 90 10 149 134 174 170 32 237 224 29 124 102 218 206 171 105 10 18 16 233 191 119 26 12 167 215 135 23 95 83 89 171 154 14 249 247 35 202 88 83 220 5 191 52 71 15 64 20 21 62 55 241 158 98 38 95 137 193 201 253 122 71 25 10 128 22 53 60 199 175 192 194 104 68 187 83 136 228 246 102 194 153 119 139 178 33 170 80 237 242 75 179 107 184 194 221 45 216 161 217 150 100 224 22 7 94 142 242 114 3 52 237 105 123 39 61 185 226 37 38 159 64 208 219 25 170 246 223 126 122 120 135 86 193 101 6 105 148 156 12 77 162 221 200 34 202 29 228 198 5 193 112 96 95 16 225 1 6 41 252 234 218 227 64 95 16 221 124 2 4 42 209 138 212 11 35 149 12 240 152 187 36 87 60 156 4 138 76 214 193 233 17 174 49 119 133 96 215 49 137 139 45 36 162 230 79 83 100 87 83 13 159 146 143 170 144 34 248 76 97 244 226 63 124 122 121 35 177 81 219 119 21 42 42 40 92 37 220 93 118 245 215 76 129 128 251 187 202 225 112 82 171 55 8 158 70 93 209 70 31 31 251 22 170 11 215 35 190 122 0 48 146 182 84 180 106 192 119 0 227 114 134 98 247 100 13 30 56 20 232 87 56 120 132 197 194 244 138 6 31 224 220 50 114 202 129 226 170 8 168 37 184 206 16 11 55 229 59 26 38 6 74 4 191 232 77 70 242 232 80 3 144 192 248 102 68 155 22 34 97 12 221 63 106 55 248 185 169 207 239 178 95 71 220 127 87 74 14 241 233 45 248 208 51 92 103 59 39 218 52 53 177 164 7 135 195 99 49 135 89 18 132 8 185 51 41 0 0 210 46 166 22 59 21 223 44 65 60 83 41 211 102 10 123 89 57 98 87 183 212 197 96 149 236]

        MAC is [104 175 109 207 180 77 52 123 170 159 77 29 255 146 246 94 95 71 63 156 183 227 45 82 160 148 152 87 53 42 161 58]
        MAC is 32 bytes, see field 15 in 3.1 on page 140 of http://public.dhe.ibm.com/security/cryptocards/pciecc4/EP11/docs/ep11-structure.pdf

        -----BEGIN HSM ENCRYPTED PRIVATE KEY-----
        AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAADsIIxQY2Y9lWElrfi1CQo
        AAAAAAAAASYAAAAAAAAAARI0md1WDAlYb+lPMgBiJnR3KrOicz1FrQIyKAXxIn1f
        1yxePMjBMi9G6j9cDJVro8f39BWRXrBkSPDxkoY+Ru68AR9biGxJbtrAz8Lvt231
        jIchlKlLcHgJrdp1tUqNYjRsbnOHcbuZ0hEbZZVs6au+2bJFmrwNT6drxd4i9loX
        UFdHICdkZCYp79KoQTp23BAHIJQ2o1ABx4ulhKHqrAdXCx9qyzL6ewmRBSibZ3sQ
        cTw90tfupJa/YcjnqhG0uFpocgpyPYHeBos4+LFuCjXqj1oKlYauqiDt4B18ZtrO
        q2kKEhDpv3caDKfXhxdfU1mrmg759yPKWFPcBb80Rw9AFBU+N/GeYiZficHJ/XpH
        GQqAFjU8x6/AwmhEu1OI5PZmwpl3i7IhqlDt8kuza7jC3S3YodmWZOAWB16O8nID
        NO1peyc9ueIlJp9A0NsZqvbffnp4h1bBZQZplJwMTaLdyCLKHeTGBcFwYF8Q4QEG
        Kfzq2uNAXxDdfAIEKtGK1AsjlQzwmLskVzycBIpM1sHpEa4xd4Vg1zGJiy0kouZP
        U2RXUw2fko+qkCL4TGH04j98enkjsVHbdxUqKihcJdxddvXXTIGA+7vK4XBSqzcI
        nkZd0UYfH/sWqgvXI756ADCStlS0asB3AONyhmL3ZA0eOBToVzh4hMXC9IoGH+Dc
        MnLKgeKqCKgluM4QCzflOxomBkoEv+hNRvLoUAOQwPhmRJsWImEM3T9qN/i5qc/v
        sl9H3H9XSg7x6S340DNcZzsn2jQ1saQHh8NjMYdZEoQIuTMpAADSLqYWOxXfLEE8
        UynTZgp7WTliV7fUxWCV7Givbc+0TTR7qp9NHf+S9l5fRz+ct+MtUqCUmFc1KqE6
        -----END HSM ENCRYPTED PRIVATE KEY-----

        -----BEGIN PUBLIC KEY-----
        MHYwEAYHKoZIzj0CAQYFK4EEACIDYgAEFOxqr0zRjjaIdwxgRM/tD89dhWXgvr/P
        /AgNbymSw271ZvYVfsaYzI6Oe1XNSUdfx5xvfTyJGGcl09feWXe18gA/pDEOByjH
        Vm+oiAj5pHX9SCGQQg9+g+YqICXSThtjBBADsIIxQY2Y9lWElrfi1CQoBCAAAAAA
        AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAQIH5LDOy8A3cQECAAAAAAAAAAB
        BBQQAQAAAAGAgAABgICAAQAKAAAAAQQgpoZCEWYIjHYfQATy6oYiSIR09Qrgrkpl
        86zfuJwzHGc=
        -----END PUBLIC KEY-----

        Data signed
        Signature verified

        ```

5. Each of the curves had a number in its name that indicates how large the key is, in bits.  We worked with 224-, 256-, 384-, and 521-bit keys  (yes that is 521 bits and not 512 bits but don't ask me why-  I first have to get smart enough to understand it myself before I get smart enough to explain it to you coherently- besides, I'm sure Google has the answer).  I am smart enough to know that there are 8 bits in a byte and while the key lengths in cryptographic algorithms are usually expressed in bits,  the lengths in our exercise output for the various blobs are expressed in bytes.  And there's nothing wrong with that, because our key blobs have lots of other metadata and attributes and fields besides just the key itself.  That's why I purposely referred to them as key blobs and not keys.

    Try this handy "one-line script" that runs this exercise four times, once for each curve, and then pipes the output to *egrep* so that we can pick out some lines of interest.  You can see how the curve strength impacts the size of the public and private key blobs:

    ``` bash
     for curve in P224 P256 P384 P521 ; do  ./lab --ex3 --curve ${curve} ; done | egrep -ie 'curve|key blob length'
    ```

    ???+ example "Expected Output"

        ```
        selected curve P224 with ObjectID of 1.3.132.0.33
        Curve ObjectID in ASN.1 BER encoding is [6 5 43 129 4 0 33]
        key pair public key blob length is 208 bytes
        key pair private key blob length is 544 bytes
        selected curve P256 with ObjectID of 1.2.840.10045.3.1.7
        Curve ObjectID in ASN.1 BER encoding is [6 8 42 134 72 206 61 3 1 7]
        key pair public key blob length is 219 bytes
        key pair private key blob length is 576 bytes
        selected curve P384 with ObjectID of 1.3.132.0.34
        Curve ObjectID in ASN.1 BER encoding is [6 5 43 129 4 0 34]
        key pair public key blob length is 248 bytes
        key pair private key blob length is 720 bytes
        selected curve P521 with ObjectID of 1.3.132.0.35
        Curve ObjectID in ASN.1 BER encoding is [6 5 43 129 4 0 35]
        key pair public key blob length is 286 bytes
        key pair private key blob length is 896 bytes
        ```

    It makes sense that the larger key sizes produce bigger key blobs.  If I ever figure out enough to give a full accounting of the difference in blob sizes, I'll update the lab.  If you ever figure it out, please create a GitHub pull request with the answer.

Let's move on to our fourth and final exercise.