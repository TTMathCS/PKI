### My notes on PKI

Note: some notes I took from the reference link below:

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

