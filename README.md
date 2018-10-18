# SoftHSM2 Docker Image

## Build and run the image

1.  Build the image

        $ docker build --tag softhsm2:2.5.0 .

2.  Run the image

        $ docker run -ti --rm softhsm2:2.5.0 sh -l

## Test it

Run the following commands within a running container.

1.  Initialise a new token

        $ softhsm2-util --init-token --slot 0 --label "My First Token"
        === SO PIN (4-255 characters) ===
        Please enter SO PIN: ****
        Please reenter SO PIN: ****
        === User PIN (4-255 characters) ===
        Please enter user PIN: ****
        Please reenter user PIN: ****
        The token has been initialized and is reassigned to slot 384541823

2.  Test the module

        $ pkcs11-tool --module /usr/local/lib/softhsm/libsofthsm2.so -l -t
        Using slot 0 with a present token (0x16eba47f)
        Logging in to "My First Token".
        Please enter User PIN: 
        C_SeedRandom() and C_GenerateRandom():
          seems to be OK
        Digests:
          all 4 digest functions seem to work
          MD5: OK
          SHA-1: OK
        Signatures: not implemented
        Verify (currently only for RSA)
          No private key found for testing
        Unwrap: not implemented
        Decryption (currently only for RSA)
        No errors

3.  Generate a new RSA keypair

        $ pkcs11-tool --module /usr/local/lib/softhsm/libsofthsm2.so -l --keypairgen --key-type rsa:2048 --id 100 --label mykey
        Using slot 0 with a present token (0x16eba47f)
        Logging in to "My First Token".
        Please enter User PIN: 
        Key pair generated:
        Private Key Object; RSA 
          label:      mykey
          ID:         0100
          Usage:      decrypt, sign, unwrap
        Public Key Object; RSA 2048 bits
          label:      mykey
          ID:         0100
          Usage:      encrypt, verify, wrap

4.  Sign a file

        # create data to sign
        $ echo "Data to sign" > data.txt

        # apply signature
        $ pkcs11-tool --module /usr/local/lib/softhsm/libsofthsm2.so --id 100 -s -m RSA-PKCS --input-file data.txt --output-file data.sig
        Using slot 0 with a present token (0x16eba47f)
        Logging in to "My First Token".
        Please enter User PIN: 
        Using signature algorithm RSA-PKCS

5.  Verify signature

        # extract pubkey
        $ pkcs11-tool --module /usr/local/lib/softhsm/libsofthsm2.so -r --id 100 --type pubkey > pubkey.der
        Using slot 0 with a present token (0x16eba47f)

        # convert pubkey to PEM
        $ openssl rsa -inform DER -outform PEM -in pubkey.der -pubin > pubkey.pem
        writing RSA key

        # verify signature
        $ openssl rsautl -verify -inkey pubkey.pem -in data.sig -pubin
        Data to sign

## References

*   [SoftHSM](https://www.opendnssec.org/softhsm)
