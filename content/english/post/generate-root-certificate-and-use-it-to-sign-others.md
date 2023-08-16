---
title: "Generate Root Certificate and Use It to Sign Others"
date: 2023-08-16T07:38:08+02:00
Description: ""
Tags: []
Categories: []
DisableComments: false
---

## Generate root certificate and use it to sign other certificates

1. Generate a Private Key for the Root CA:

Generate a private key that will be used to sign your root CA certificate. Here's an example command to generate a 2048-bit RSA private key:

```sh
openssl genpkey -algorithm RSA -out root.key
```

2. Generate a Self-Signed Root CA Certificate:

Use the private key generated in the previous step to create a self-signed root CA certificate. You'll typically need to provide information for the certificate's fields like Common Name (CN), Organization (O), etc. Here's an example command:

```sh
openssl req -x509 -new -key root.key -out root.crt
```

3. Follow the prompts to fill in the certificate information.

Add Root CA Certificate to Trust Store:
To make your system trust the root CA, you need to add its certificate to your system's trust store. The process for this varies depending on your operating system.

* Windows: Import the root.crt file into the "Trusted Root Certification Authorities" store using the Windows Certificate Manager (certmgr.msc).
* macOS: Import the root.crt file into the "System" keychain using the Keychain Access app and mark it as trusted.
* Linux: The process can vary depending on the distribution, but generally, you would need to place the root.crt file in a trusted certificate directory and update the certificate trust settings.

Now, any certificate signed by the root CA you generated will be trusted by your system.

Please note that this root certificate should be kept secure, as it has the authority to sign other certificates. It's also important to mention that while this process creates a self-signed root CA, in production environments, you would obtain root CA certificates from trusted third-party certificate authorities. This process is mainly for local development and testing purposes.

## Create and sign a new certificate using root CA

To sign a certificate with the root CA you generated, you need to create a Certificate Signing Request (CSR) for the certificate you want to generate, and then sign that CSR using your root CA's private key. Here's how you can do it using OpenSSL:

1. Generate a Private Key for the Certificate:

Generate a private key for the certificate you want to create:

```sh
openssl genpkey -algorithm RSA -out server.key
```

2. Create a CSR for the Certificate:

Generate a Certificate Signing Request (CSR) using the private key you just generated:

```sh
openssl req -new -key server.key -out server.csr
```

You'll be prompted to provide information for the certificate fields.

3. Sign the CSR with the Root CA:

Sign the CSR using the root CA's private key and certificate:

```sh
openssl x509 -req -in server.csr -CA root.crt -CAkey root.key -CAcreateserial -out server.crt -days 365
```
* -CA root.crt: Specifies the root CA certificate.
* -CAkey root.key: Specifies the root CA's private key.
* -CAcreateserial: Creates a serial number file to keep track of signed certificates.
* -out server.crt: Specifies the output file for the signed certificate.
* -days 365: Sets the validity period of the signed certificate.

4. Use the Signed Certificate and Key:

The server.crt file is your signed certificate, and the server.key file is the private key associated with it. You can use these files in your server configuration for secure communication.

Remember that this process generates a self-signed certificate chain. For production use, you would typically use the root certificate provided by a trusted certificate authority to sign your server's certificate, creating a chain of trust that is recognized by common web browsers and devices. This process is mainly for local development and testing purposes.

## Generate .pfx file to use in ASP.NET Core

The equivalent OpenSSL command for generating a PKCS12/PFX file like the mkcert command you provided would involve a series of steps, as OpenSSL provides a more flexible and comprehensive toolkit for cryptographic operations. The mkcert command you provided generates a PKCS12 file (.pfx) that includes both the private key and the corresponding certificate. Here's how you could achieve a similar result using OpenSSL:

```sh
# Combine the private key and the certificate into a PKCS12/PFX file
openssl pkcs12 -export -out server.pfx -inkey root.key -in root.crt
```

Explanation of the commands:

1. The first command generates a private key (local.app.key) and a certificate signing request (local.app.csr).
2. The second command generates a self-signed certificate (local.app.crt) using the previously generated CSR and private key.
3. The third command combines the private key and the certificate into a PKCS12/PFX file (local.app.pfx).

Please note that OpenSSL provides many options for customization, so you might want to adjust the parameters (such as key size, validity period, and certificate extensions) according to your requirements. Also, remember that while the above commands generate a self-signed certificate, in a production environment, you would typically get certificates signed by a trusted certificate authority.