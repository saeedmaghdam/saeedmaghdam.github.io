---
title: "Load Certificates in Asp Net Core"
date: 2023-08-14T00:57:33+02:00
Description: ""
Tags: []
Categories: []
DisableComments: false
---

## Load certificates in asp.net core application

To convert your server's certificate and private key files (server.crt and server.key) into a PFX (Personal Information Exchange) format, which is a format that combines both the certificate and the private key into a single file, you can use OpenSSL. Here's how to do it:

1. Convert the Certificate and Private Key to PEM Format (if needed):

If your server.crt and server.key files are not already in PEM format, you can convert them using the following commands:

```ssh
openssl x509 -in server.crt -outform PEM -out server.pem
openssl rsa -in server.key -outform PEM -out server_key.pem
```

These commands will create server.pem (converted certificate) and server_key.pem (converted private key) files.

2. Combine the Certificate and Private Key into a PFX File:

Now that you have the certificate and private key in PEM format, you can combine them into a PFX file using the following command:

```ssh
openssl pkcs12 -export -out server.pfx -inkey server_key.pem -in server.pem
```

This command will prompt you to set a password for the PFX file. Make sure to remember this password, as you'll need it when using the PFX file later.

After running these commands, you should have a server.pfx file that contains both the certificate and private key in a single file. This PFX file can be used in various server setups, such as importing it into web servers like IIS, Apache, or nginx, or using it in other applications that require the combined certificate and private key.

Remember to keep the PFX file secure, as it contains sensitive information. Additionally, ensure that you follow best practices for managing certificates and keys, such as securely storing them and regularly renewing them as needed.

3. Load the certificate from asp.net core:

```cs
    builder.WebHost.ConfigureKestrel((context, serverOptions) =>
    {
        serverOptions.Listen(IPAddress.Any, 5000);
        serverOptions.Listen(IPAddress.Any, 5001, listenOptions =>
        {
            listenOptions.UseHttps("server.pfx", "test");
        });
    });
```

#### Resources:
* [Configure endpoints for the ASP.NET Core Kestrel web server](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/servers/kestrel/endpoints?view=aspnetcore-7.0)