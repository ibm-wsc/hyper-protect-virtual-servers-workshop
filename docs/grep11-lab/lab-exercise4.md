# Exercise 4 - Wrap and Unwrap a Key

## Overview of Exercise 4

Data that is not encrypted is referrd to as plaintext.  Data that is encrypted is referred to as ciphertext.

Whether this data is human readable text, like this sentence, or binary data (like a lot of the output in our exercises!), or whatever, is usually of little concern to encryption and decryption algorithms.  The input is just a bunch of bits- either *1* or *0*. 

Keys themselves are just a bunch of bits- *1*'s and *0*'s.  Can you encrypt and decrypt keys with other keys?  You sure can. There's nothing magical about that. But, like all disciplines, cryptographers have special terms for encrypting and decrypting keys- they use the term *wrapping* a key to refer to encrypting a key, and *unwrapping* a key refers to decrypting a key. There are valid reasons to make the distinction- for example, the PKCS #11 standard allows you to assign attributes to keys, and you can create keys such that they can or cannot wrap or unwrap keys, and you may wish to create different keys for different purposes. In fact, it is considered best practice to not use the same key to encrypt both data and keys, so using different terminology- *encrypting data* versus *wrapping keys* allows for easier separation of duties and may make key management easier.

In this exercise, the following use case will be demonstrated- securely transmitting a secret key so that both parties have the same secret key and can use it for encryption during a session:

1. Sender wishes to send a secret (i.e., symmetric) key to a recipient.  
2. Sender generates a secret key and wraps it with the recipient's public key
3. Sender sends the wrapped key to the recipient-  this is safe since it is wrapped with the recipient's public key
4. Recipient receives the wrapped key, and unwrarps it with her private key

## Run Exercise 4

1.  Run the fourth exercise:

    ``` bash
    ./lab --ex4
    ```

    ??? example "Example Output"

        ```
        Generated AES key
        Generated PKCS RSA key pair

        the AES key prior to being wrapped is:
        [0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 3 176 130 49 65 141 152 246 85 132 150 183 226 212 36 40 0 0 0 0 0 0 141 37 0 0 0 0 0 0 0 1 18 52 228 123 210 48 148 72 67 55 62 73 210 121 66 247 116 178 245 24 106 221 115 156 71 47 218 46 103 246 124 74 119 150 39 244 103 45 22 196 216 35 98 215 63 143 111 10 217 35 1 34 26 127 84 98 120 216 45 88 92 129 231 135 244 57 118 125 8 141 119 168 29 141 122 29 224 59 1 244 174 182 103 220 131 195 114 27 143 94 112 118 185 49 140 147 51 10 26 16 150 95 130 1 239 21 212 148 246 182 10 159 243 69 233 30 202 254 52 241 146 79 66 44 9 199 32 56 113 169 227 133 51 177 98 204 240 57 244 188 254 99 38 243 227 153 119 170 28 38 109 86 197 175 193 64 255 113 106 40 94 90 204 5 241 135 72 79 76 42 166 180 59 93 91 62 94 248 153 138 201 11 2 155 224 41 66 230 89 253 94 74]

        the checksum of the AES key is: [147 241 235]

        the AES key has been wrapped with a PKCS RSA Public Key
        The AES key has been unwrapped with the PKCS Private Key corresponding to the PKCS Public Key we used to wrap it with.

        the AES key after being wrapped and unwrapped is:
        [0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 3 176 130 49 65 141 152 246 85 132 150 183 226 212 36 40 0 0 0 0 0 0 12 0 0 0 0 0 0 0 0 1 18 52 206 249 87 66 227 239 81 119 120 21 113 29 123 27 2 52 241 6 31 39 195 183 118 39 98 185 255 179 31 44 217 72 38 56 126 190 23 230 112 120 180 74 27 118 70 9 145 247 109 197 15 92 126 232 161 192 140 63 241 178 29 188 70 79 29 240 6 39 76 22 244 149 99 241 220 160 157 40 58 42 255 121 3 181 9 118 15 96 170 49 208 117 127 248 187 202 226 92 76 198 142 164 39 85 247 74 203 28 50 106 162 144 217 93 110 120 157 126 106 37 202 133 10 133 162 89 201 186 210 12 93 124 74 231 120 172 149 134 196 232 247 5 228 92 146 119 245 115 107 124 153 218 87 65 119 80 131 101 53 219 210 111 107 216 117 6 45 153 176 221 49 121 208 77 164 68 230 219 190 45 238 82 201 112 126 205 97 110 201 187 28 240 134 33 223 206 209 45 207 159 132 178 222 232 63 160]

        the checksum of the unwrapped AES key is: [147 241 235 0 0 1 0]

        ```

    I mentioned in the overview of this exercise that you can assign attributes to keys when you create them. The code for this exercise, by default, assigns an attribute that allows the public key it creates to wrap keys, and it assigns an attribute to the private key to unwrap keys. 

2. Run this command which will cause the program to set the applicable attribute for the private key such that it is not allowed to unwrap a key. Observe in the output that, while the wrap operation proceeds, the unwrap fails:

    ``` bash
    ./lab --ex4 --unwrap=false
    ```

    ??? example "Example output"
    
        ```
        Generated AES key
        Generated PKCS RSA key pair

        the AES key prior to being wrapped is:
        [0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 3 176 130 49 65 141 152 246 85 132 150 183 226 212 36 40 0 0 0 0 0 0 141 37 0 0 0 0 0 0 0 1 18 52 120 27 243 16 225 139 153 149 120 188 137 72 50 120 131 243 65 98 62 77 65 60 178 184 93 56 192 2 202 179 211 210 240 252 246 171 56 100 172 203 61 233 216 102 186 60 207 208 1 173 188 223 4 45 199 93 159 251 80 32 171 131 10 228 235 131 98 191 70 58 116 218 179 225 62 235 136 76 246 39 55 21 247 126 181 47 3 130 64 70 198 91 136 21 133 177 138 190 118 139 183 195 32 110 107 230 101 253 127 40 171 124 1 146 32 138 11 148 175 180 54 21 123 51 144 234 167 228 218 128 47 74 26 203 250 95 110 208 47 245 229 198 164 121 76 226 138 122 31 168 169 251 74 149 174 62 88 86 122 204 149 195 200 41 139 250 0 242 226 34 193 78 32 151 250 220 249 249 106 106 231 217 34 139 118 29 206 140 217 75]

        the checksum of the AES key is: [79 136 190]

        the AES key has been wrapped with a PKCS RSA Public Key
        panic: Unwrap AES key error: rpc error: code = Unknown desc = CKR_KEY_FUNCTION_NOT_PERMITTED

        goroutine 1 [running]:
        main.wrapAndUnwrapKey(0xc420030001)
                /home/hyper-protect-lab/go/src/github.com/ibm-developer/ibm-cloud-hyperprotectcrypto/golang/lab/exercise4.go:110 +0x1543
        main.main()
                /home/hyper-protect-lab/go/src/github.com/ibm-developer/ibm-cloud-hyperprotectcrypto/golang/lab/main.go:79 +0x35e
        ```

3. This command will cause the public key to be created without the ability to wrap a key. The command will fail when trying to wrap the key:

    ``` bash
    ./lab --ex4 --wrap=false
    ```

    ??? example "Example output"

        ```
        Generated AES key
        Generated PKCS RSA key pair

        the AES key prior to being wrapped is:
        [0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 3 176 130 49 65 141 152 246 85 132 150 183 226 212 36 40 0 0 0 0 0 0 141 37 0 0 0 0 0 0 0 1 18 52 94 240 28 56 111 164 179 172 71 8 202 169 163 165 249 244 192 122 26 215 148 231 8 62 1 127 152 148 45 31 45 189 172 0 97 131 35 17 231 104 88 103 196 2 96 167 213 198 9 62 10 241 154 128 94 206 4 229 36 89 125 38 231 94 155 212 43 90 44 253 221 170 164 83 13 228 232 156 205 205 183 65 252 45 183 191 75 243 56 130 64 170 140 197 150 91 224 229 88 47 185 97 138 145 93 241 181 206 4 141 225 214 249 62 21 4 83 105 12 209 109 187 124 171 112 222 107 44 203 204 240 23 236 60 27 190 248 201 189 3 112 89 163 28 91 254 220 95 74 11 247 137 54 107 93 106 21 199 166 152 208 200 151 220 233 4 125 105 139 235 83 211 121 166 15 209 229 173 92 2 74 50 107 41 73 150 197 116 162 37]

        the checksum of the AES key is: [229 2 226]

        panic: Wrap AES key error: rpc error: code = Unknown desc = CKR_KEY_FUNCTION_NOT_PERMITTED

        goroutine 1 [running]:
        main.wrapAndUnwrapKey(0xc420030100)
                /home/hyper-protect-lab/go/src/github.com/ibm-developer/ibm-cloud-hyperprotectcrypto/golang/lab/exercise4.go:90 +0x1553
        main.main()
                /home/hyper-protect-lab/go/src/github.com/ibm-developer/ibm-cloud-hyperprotectcrypto/golang/lab/main.go:79 +0x35e
        ```

    !!! note
        The public key has been prevented from wrapping a key, but it could still be used to encrypt data.

    Congratulations for reaching the end of the lab!
