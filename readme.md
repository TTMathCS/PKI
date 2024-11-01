<!-- TOC -->
* [Table of Content](#table-of-content)
    * [PKI Introduction](#pki-introduction)
    * [The Goal of certificate and PKI: to bind name to public keys](#the-goal-of-certificate-and-pki-to-bind-name-to-public-keys)
    * [PKI related terms](#pki-related-terms)
    * [MACs and signatures authenticate stuff](#macs-and-signatures-authenticate-stuff)
    * [What key pair can do?](#what-key-pair-can-do)
    * [Certificate format](#certificate-format)
    * [x.509 certificate structure (V3 defined in RFC5280)](#x509-certificate-structure-v3-defined-in-rfc5280)
    * [What is ASN.1 (Abstract Syntax Notion one)?](#what-is-asn1-abstract-syntax-notion-one)
    * [Certificate Binary Formats:](#certificate-binary-formats)
    * [Public-key cryptography](#public-key-cryptography)
    * [RSA (Prime Number + Modular arithmatic)](#rsa-prime-number--modular-arithmatic)
      * [Algo to find Prime Numbers as below, but no good methods to break them up](#algo-to-find-prime-numbers-as-below-but-no-good-methods-to-break-them-up)
      * [How it works](#how-it-works)
    * [ECDSA (Elliptic Curve Digital Signature Algorithm)](#ecdsa-elliptic-curve-digital-signature-algorithm)
    * [TrustStores and KeyStores](#truststores-and-keystores)
    * [Commands to create self-signed certificate chain](#commands-to-create-self-signed-certificate-chain)
    * [Commands that import certificate into TrustStore (JKS format)](#commands-that-import-certificate-into-truststore-jks-format)
    * [Commands that import x.509 certificate and private key into Java Keystore](#commands-that-import-x509-certificate-and-private-key-into-java-keystore)
    * [Commands to view a certificate file](#commands-to-view-a-certificate-file)
    * [Commands to view/export certificate of remote server](#commands-to-viewexport-certificate-of-remote-server)
<!-- TOC -->

---

### PKI Introduction

There are so many PKI terms and resources you can search on web, but unfortunately so many confused me and there looks
no easy and simple answers.
Below is the outline of what I summarize of PKI

1. **What information** should be included in a certificate? X.509 defines what data can go into certificate, but at
   high
   level of metadata.
2. **What abstract format** should x.509 use? It is like when save data, should I use json or yaml? ASN.1 defines the
   abstract layer of format of X.509 object
3. **What binary format** should a certificate be saved? You can use text file, word document or Apple pages to save
   documents, which will save the data with different binary format. For certificate, popular one are der, pem, pkcs#7
   and pkcs#12.

Above are summary of those terms and relations. If you are not bored enough you may continue the reading of below.

reference link:

- [Everything you should know about certificates and PKI but are too afraid to ask](https://smallstep.com/blog/everything-pki/)
- [Understanding X.509 digital certificate thumbprints](https://morgansimonsen.wordpress.com/2013/04/16/understanding-x-509-digital-certificate-thumbprints/)

---

### The Goal of certificate and PKI: to bind name to public keys

The rest is just implementation details.

---

### PKI related terms

- entity
- identity
- identifier
- name
- claim
- subscriber (end entity)
- certificate authority (CA) - issuer
- leaf certificates
- root certificate
- intermediate certificate
- relying party

---

### MACs and signatures authenticate stuff

MAC = Message Authentication Code

---

### What key pair can do?

- Encrypt data with a public key, which can only be decrypted by private key
- Sign some data with private key, which can be verified by public key

---

### Certificate format

- X.509 v3 (PKIX described in RFC 5280), used for HTTPS over TLS if certificate is used without additional qualification
- SSH
- PGP

---

### x.509 certificate structure (V3 defined in RFC5280)

X.509 is a standard of certificate. it defines which data should go into certificate, and are just objects with the name
of the servers, the name of who signed the certificates, the signature and so on as below.

- version number
- serial number
- Signature Algorithm ID
- Issuer Name
- Validity period
    - Not Before
    - Not After
- Subject name
- Subject Public Key Info
    - Public Key Algorithm
    - Subject Public Key
- Issuer Unique Identifier (Optional)
- Subject Unique Identifier (Optional)
- Extension (Optional)
    - ...
- Certificate Signature Algorithm
- Certificate Signature

Command to check a website certificate

```shell
# with SNI
openssl s_client -showcerts -servername www.google.ca -connect www.google.ca:443 </dev/null \
| openssl x509 -inform pem -text

# without SNI
openssl s_client -showcerts -connect www.google.ca:443 </dev/null \
| openssl x509 -inform pem -text

# check all cert in pem file
while openssl x509 -noout -text; do :; done < _.google.com.pem
```

---

### What is ASN.1 (Abstract Syntax Notion one)?

How should we write our certificate in a computer format? There are a billion ways formatting a document and if we don't
agree on one, then we will never be able to ask a computer to parse a x509 certificate. ASN.1 tells you exactly how you
should write your object/certificate, like a format of json or yaml, but which don't defines how they were saved as
bits/bytes (those are defined by SSL formats, like der, pem, pkcs# and etc.)

ASN.1 is the metadata definition on what data needs to be included, mandatory or optional, but it doesn't define the
actual storage format saved.
[TLDR.,](https://morgansimonsen.wordpress.com/2013/04/16/understanding-x-509-digital-certificate-thumbprints/)

X.509 builds on ASN.1, another ITU-T standard (defined by X.208 and X.680). ASN stands for
Abstract Syntax Notation (1 stands for One). ASN.1 is a notation for defining data types.
You can think of it like JSON for X.509 but it's actually more like protobuf or thrift or SQL
DDL. RFC 5280 uses ASN.1 to define an X.509 certificate as an object that contains various
bits of information: a name, key, signature, etc.

ASN.1 is abstract in the sense that the standard doesn't say anything about how stuﬀ
should be represented as bits and bytes. For that there are various encoding rules that
specify concrete representations for ASN.1 data values. It's an additional abstraction layer
that's supposed to be useful, but is mostly just annoying. It's sort of like the diﬀerence
between unicode and utf8 (eek).

There are a bunch of encoding rules for ASN.1, but there's only one that's commonly used
for X.509 certificates and other crypto stuﬀ: distinguished encoding rules or DER (though
the non-canonical basic encoding rules (BER) are also occasionally used). DER is a pretty
simple type-length-value encoding, but you really don't need to worry about it since
libraries will do most of the heavy lifting.


---

### Certificate Binary Formats:

- X.509 defines what data go into certificates
- ASN.1 defines what abstract format it should have (like json or yaml)
- SSL format decides how to convert the certificate into bits and bytes. (PEM, PKCS7, DER, and PKCS#12)

TLDR., https://comodosslstore.com/resources/a-ssl-certificate-file-extension-explanation-pem-pkcs7-der-and-pkcs12/

- **DER** (Distributed Encoding Rule): a binary form, can include certificates and private keys of all types. They
  mostly use .cer or .der extensions. The DER certificate format is mostly
- **PEM** (Private Enhanced Mail): contains Base64 encoded data of der. Extensions can be .pem, .crt, .cer, or .key
  formats.
  A PEM certificate file may consists of the server certificate and intermediate certificate in a separate .crt or .cer
  file and the private key is in a .key file.
    - certificate: It starts with "-----BEGIN CERTIFICATE-----" and ends with "-----END CERTIFICATE-----"
    - private key: start with "-----BEGIN PRIVATE KEY-----" and ends with "-----END PRIVATE KEY-----"
    - CSR: starts with "-----BEGIN CERTIFICATE REQUEST-----" and ends with "-----END CERTIFICATE REQUEST-----"
- **P7B/PKCS#7**: encoded in Base64 ASCII encoding and usually have .p7b or .p7c file extensions. The thing that
  separates
  PKCS#7 formatted certificates is that only certificates can be stored in this format, not private keys. In other
  words, a P7B file will only consist of certificates and chain certificates. The certificates having P7B/PKCS#7 format
  are contained between the “—–BEGIN PKCS7—–” and “—–END PKCS7—–” statements. Microsoft Windows and Java Tomcat are the
  most common platforms using this format for SSL certificates.
- **PFX/P12/PKCS#12**:  all of which refer to a personal information exchange format — is the binary format that stores
  the server certificate, the intermediate certificate and the private key in a single password-protected pfx or .p12
  file. These files are typically used on Windows platforms i to allow you to import and export certificates and private
  keys.
- Basic Encoding Rules (BER)
- Canonical Encoding Rules (CER)
- XML Encoding Rules (XER)
- Canonical XML Encoding Rules (CXER)
- Extended XML Encoding Rules (E-XER)
- Packed Encoding Rules (PER, unaligned: UPER, canonical: CPER)
- Generic String Encoding Rules (GSER)

---

### Public-key cryptography

https://en.wikipedia.org/wiki/Public-key_cryptography

- RSA (Rivest–Shamir–Adleman)
- ECDSA
- Ed25519
- DSA
- Diffie-Hellman
- AES
- SHA

---

### RSA (Prime Number + Modular arithmatic)

RSA is used to generate private key and public key

Refs:

- https://www.youtube.com/watch?v=qph77bTKJTM

The security of RSA relies on the practical difficulty of factoring the product of two large prime numbers, the "
factoring problem". Breaking RSA encryption is known as the RSA problem. Whether it is as difficult as the factoring
problem is an open question. There are no published methods to defeat the system if a large enough key is used.

#### Algo to find Prime Numbers as below, but no good methods to break them up

- Mersenne primes
- Fermat primes
- Pocklington primality test
- Baillie-BSW primality test
- Miller-Rabin primality test
- Sieve of Eratosthenes

#### How it works

1. pick up 2 prime number p and q, $` n = p * q`$
2. pick number e, e is co-prime to $` (p-1)(q-1) `$
3. m is the message to encrypt. c = $`m^e (mod \ n) `$
4. calculate d that $` d*e = 1 \ (mod \ (p-1)(q-1)) `$
5. decrypt: $` d = c^d \ (mod \ n) `$

![img](resources/RSA_Algorithm.png)

**Question 1: Is it possible to pre-calculate** all those prime numbers and save them in a rainbow table for fast
lookup?

**Answer:** The Prime Numbers (Semi-PrimeNumbers) are randomly picked during key pair generation, if they pass the
primality
tests, they will be picked, otherwise it keeps trying until a big prime number is found. They are 2^1024 or 2^2048
big so that it is impossible to find all the prime numbers, or even save all those prime numbers (no such big storage).

---

### ECDSA (Elliptic Curve Digital Signature Algorithm)

ECDSA is a Digital Signature Algorithm (DSA) which uses keys derived from ECC(Elliptic Curve Cryptography). It is a
particularly efficient equation based on PKC (Public Key Cryptography).
EdDSA is a digital signature scheme using variant of Schnorr signature based on twisted Edwards Curves.
It is designed to be faster than existing digital signature schemes without sacrificing security.

**Ed25519** is the EdDSA signature scheme using SHA-512 and Curve25519 where:

$` q = 2^{255} - 19 `$

Twisted Edwards Curve: $` -x^2 + y^2 = 1 - \frac{121665}{121666}x^2y^2`$

TLDR, https://en.wikipedia.org/wiki/EdDSA#Ed25519

---

### TrustStores and KeyStores

They are mostly used for JAVA. They can be JKS keystore type, or PCKS12 keystore type.

- A KeyStore consists of a database containing a private key and an associated certificate, or an associated certificate
  chain. The certificate chain consists of the client certificate and one or more certification authority (CA)
  certificates.
- A TrustStore contains only the certificates trusted by the client (a “trust” store). These certificates are CA root
  certificates, that is, self-signed certificates.

TLDR,: https://www.ibm.com/docs/ro/zos-connect/zosconnect/3.0?topic=connect-keystores-truststores


--- 

### Commands to create self-signed certificate chain

```shell
# make certs for root ca
openssl genrsa -out root.key 2048
openssl req -x509 -sha256 -nodes -extensions v3_ca -key root.key -subj "/C=CA/ST=ON/O=HelloWorld/CN=root.example.com" -days 3650 -out root.crt

# make certs for intermediate ca
openssl genrsa -out intermediate.key 2048
openssl req -new -sha256 -nodes -key intermediate.key -subj "/C=CA/ST=ON/O=HelloWorld/CN=intermediate.example.com" -out intermediate.csr
openssl x509 -req -extensions v3_ca -in intermediate.csr -CA root.crt -CAkey root.key -CAcreateserial -out intermediate.crt -days 500 -sha256

# make certs for end user
openssl genrsa -out enduser.key 2048
openssl req -new -sha256 -nodes -key enduser.key -subj "/C=CA/ST=ON/O=HelloWorld/CN=enduser.example.com" -out enduser.csr
openssl x509 -req -in enduser.csr -CA intermediate.crt -CAkey intermediate.key -CAcreateserial -out enduser.crt -days 500 -sha256

# verify 
openssl verify -CAfile <(cat intermediate.crt root.crt) enduser.crt
openssl verify -CAfile <(cat root.crt intermediate.crt) enduser.crt
```

---

### Commands that import certificate into TrustStore (JKS format)

```shell
# import test.crt into existing java truststore. 
#The "keytool -importcert" command had no trouble reading the certificate in both PEM and DER formats.
keytool -importcert -file <openssl_crt.pem> -keystore <jks-file-name.jks> -storepass jkspass -alias <alias-name> -keypass <keypass>
keytool -trustcacerts -keystore "/jdk/jre/lib/security/cacerts" -storepass changeit -importcert -alias testalias -file "/opt/ssl/test.crt"

# create new truststore and import the certificate
keytool -import -alias testalias -file test.crt -keypass keypass -keystore test.jks -storepass test@123

```

---

### Commands that import x.509 certificate and private key into Java Keystore

Believe or not, keytool does not provide such basic functionality like importing private key to keystore. You can try
this workaround with merging PKSC12 file with a private key to a keystore

```shell
#Step one: Convert the x.509 cert and key to a pkcs12 file
#Note: Make sure you put a password on the pkcs12 file - otherwise you'll get a null pointer exception when you try to import it. 
openssl pkcs12 -export -in server.crt -inkey server.key -out server.p12 -name [some-alias] -CAfile ca.crt -caname root

#Step two: Convert the pkcs12 file to a Java keystore
keytool -importkeystore \
        -deststorepass [changeit] -destkeypass [changeit] -destkeystore server.keystore \
        -srckeystore server.p12 -srcstoretype PKCS12 -srcstorepass some-password \
        -alias [some-alias]
```

---

### Commands to view a certificate file
```shell
# view the contents of a .pem certificate
openssl x509 -in certificate.crt -text -noout
openssl x509 -inform pem -noout -text -in 'cerfile.cer'
openssl x509 -inform der -noout -text -in 'cerfile.cer'
keytool -printcert -file certificate.pem

# windows
certutil -dump C:\path\certfile.cer

#Using `openssl` to display all certificates of a PEM file
while openssl x509 -noout -text; do :; done < cert-bundle.pem

# view the contents of a der format certificate
openssl x509 -inform der -in CERTIFICATE.der -text -noout


# list items in truststore/keystore
keytool -v -list -keystore /path/to/keystore
keytool -list -keystore /path/to/keystore -alias foo

```


### Commands to view/export certificate of remote server
```shell
# view cert of a remote server
echo | openssl s_client -showcerts -servername gnupg.org -connect gnupg.org:443 2>/dev/null | openssl x509 -inform pem -noout -text

#shows the chain (as served) with nearly all details in a mostly rather ugly format
keytool -printcert -sslserver $host[:$port]

# export remote server certificate
curl --insecure -vvI https://www.example.com 2>&1 | awk 'BEGIN { cert=0 } /^\* SSL connection/ { cert=1 } /^\*/ { if (cert) print }'
openssl s_client -showcerts -connect host.name.com:443 -servername host.name.com  </dev/null | sed -ne '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p' > host.name.com.pem

#you may also convert to a certificate for desktop
openssl x509 -inform PEM -in host.name.com.pem -outform DER -out host.name.com.cer


```
