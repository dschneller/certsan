# certsan: Certificate Santiy Checker

This is a little helper script I wrote to help me verify that certificates
and private keys I get sent by other people actually fit together. Especially
with people unfamiliar with certificates and cryptography, mixups are easy
and easy to miss.

More than once I noticed that a private key was not the right one for
a given certificate only when e. g. haproxy complained about it.

## Prerequisites
This was written on a Mac, with Ruby 2.0. I do not see why it would not work
on any other platform as well, but it has never been tested.
You also need the OpenSSL binary `openssl` on your search path.

## What does it do?
The script will use OpenSSL to get the modulus from the given key, certificate
and optionally signing request and compare them. If one of them does not match
the other(s), the script will exit with a non-zero return code. In verbse mode
you get MD5 hashes of the moduli it found.

## Usage
Key and certificate files are mandatory. The CSR is optional. If it is provided
it must match the key and certificate modulus.

```
➜  certsan -v
```
```
Usage: certsan OPTIONS
    -c, --crt CRTFILE                Certificate (CRT) file (path)
    -k, --key KEYFILE                Key file (path)
    -r, --csr CSRFILE                [Optional] Certificate Signing Request (CSR) file (path)
    -v, --verbose                    Display more stuff.

Common options:
    -h, --help                       Show this message
```

## Sample output

### Failed Check
```
➜  certsan -v -c www.example.com.crt -k www.example.com.key -r www.example.com.csr
```
```
Certificate Modulus (md5): 5b22d155d7392ccdeb02532eeb5f9733
Key Modulus         (md5): e9bd187b826442e02632e548df3e10a9
CSR Modulus         (md5): 5b22d155d7392ccdeb02532eeb5f9733
SANITY CHECK FAILED!
```
The output shows that the certificate was apparently issued based on the provided CSR, but
the private key file does not match the certificate.

### Successful check
```
➜  certsan -v -c www.example.com.crt -k www.example.com.key -r www.example.com.csr
```
```
Certificate Modulus (md5): 1df8459f2aae9698e8ee9373d136ccbc
Key Modulus         (md5): 1df8459f2aae9698e8ee9373d136ccbc
CSR Modulus         (md5): 1df8459f2aae9698e8ee9373d136ccbc
SANITY CHECK OK
```

## Limits
The files are passed to the OpenSSL `x509`, `rsa` and `req` commands respectively. They
must be parseable by these commands.

