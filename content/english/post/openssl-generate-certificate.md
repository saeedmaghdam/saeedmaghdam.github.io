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

## Install certificates in Dockerfile

Alpine:

```sh
RUN apk update && apk add --no-cache ca-certificates grep netcat-openbsd iputils iproute2

COPY certs/certificate.crt /usr/local/share/ca-certificates/certificate.crt
RUN update-ca-certificates
```

Debian:
```sh
RUN apt-get update && apt-get install -y ca-certificates grep netcat-openbsd iputils-ping iproute2

COPY certs/certificate.crt /usr/local/share/ca-certificates/certificate.crt
RUN update-ca-certificates
```

## Test Certificates

To check if certificates are installed successfully on a Linux system, you can use various commands and tools to verify the presence of the certificates in the appropriate certificate store and to ensure that they are trusted. Here are a few ways to check if certificates are installed:

1. Check Certificate Files:

You can manually navigate to the directory where the certificates are supposed to be installed and verify their presence:

```sh
ls /usr/local/share/ca-certificates/
```

This command will list the certificate files in the specified directory.

2. Check Certificate Store:

After copying the certificate files, you can use the update-ca-certificates command to update the certificate store. If this command runs successfully, it means the certificates have been added to the system's trusted certificate store.

```sh
update-ca-certificates --fresh
```

3. Verify with OpenSSL:

You can use the openssl command-line tool to verify the certificates. For instance, you can use the following command to check the validity of a certificate:

```sh
openssl x509 -in /usr/local/share/ca-certificates/certificate.crt -noout -text
```

This command will display detailed information about the certificate, including its validity period.

4. Verify Using cURL:

You can use cURL to test connections to servers using the certificates. For example:

```sh
curl --cacert /usr/local/share/ca-certificates/certificate.crt https://example.com
```

Replace https://example.com with a valid URL that requires SSL/TLS verification. If the connection succeeds, it means the certificate is trusted.

5. Check for Errors in Applications:

If you're using certificates for specific applications, you can run those applications and monitor their logs for any certificate-related errors or warnings. Most applications will log errors if they encounter issues with certificates.

Remember that the effectiveness of these checks depends on the context in which you're using the certificates. Make sure you're using the correct paths and filenames in the commands, and adapt the checks to your specific setup and requirements.