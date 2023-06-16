---
title: "Openssl Generate Certificate"
date: 2023-06-16T14:20:17+02:00
Description: ""
Tags: []
Categories: []
DisableComments: false
---

## Install OpenSSL

Visit one of the following websites and download the proper version of SSL for your operating system:
* [https://wiki.openssl.org/index.php/Binaries](https://wiki.openssl.org/index.php/Binaries)
* [https://slproweb.com/products/Win32OpenSSL.html](https://slproweb.com/products/Win32OpenSSL.html)

After installing, check if it is installed:

```sh
openssl version
```

## Generating a certificate

1. Generate a private key file and password-protected certificate signing request (CSR) by running the following command:
```sh
openssl req -newkey rsa:2048 -nodes -keyout private.key -out csr.csr
```

This command generates two files:
* private.key: The private key file.
* csr.csr: The certificate signing request file.

Follow the prompts to enter the requested information, such as country, organization, common name, etc.

2. Generate a self-signed certificate using the private key and CSR by running the following command:
```sh
openssl x509 -req -sha256 -days 365 -in csr.csr -signkey private.key -out certificate.crt
```

This command generates the certificate.crt file, which is the self-signed certificate.

3. Optionally, you can convert the certificate and private key to PKCS#12 format with a password by running the following command:

```sh
openssl pkcs12 -export -out certificate.pfx -inkey private.key -in certificate.crt -password pass:YOUR_PASSWORD
```

You now have a self-signed certificate and, optionally, a PKCS#12 file with a password.

> Remember to keep the private key file (private.key) and the PKCS#12 file (certificate.pfx) secure and protected, as they contain sensitive information.

Please note that self-signed certificates are not trusted by default by most systems and web browsers. For production use, you would typically obtain a certificate from a trusted certificate authority (CA).