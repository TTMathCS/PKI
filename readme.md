### My notes on PKI

Note: some notes I took from the reference link below:
- [Everything you should know about certificates and PKI but are too afraid to ask](https://smallstep.com/blog/everything-pki/)
- [Understanding X.509 digital certificate thumbprints](https://morgansimonsen.wordpress.com/2013/04/16/understanding-x-509-digital-certificate-thumbprints/)

---

### The Goal of certificate and PKi: to bind name to public keys
The rest is just implementation details.

---

### Broad overview and background information
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

### x.509 certificate structure
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
