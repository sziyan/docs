---
title: Certificate Commands Cheatsheet
tags: 
    - tech
    - homelab 
---


## Convert p7b to cer
- Normally used when CA provided a signed certificate
- Generated certificate bundle contains signed certificate as well as rootCA and intermediate certs
```
openssl pkcs7 -in certificate.p7b -print_certs > bundle.cer
```
## Convert cert to pem file
```
openssl x509 -in certificate.crt -outform PEM -out certificate.pem
```

## Extract private key from pfx 
```
openssl pkcs12 -in yourfile.pfx -nocerts -nodes -out privateKey.key
```

## Convert cer to crt
```
openssl x509 -in yourfile.cer -out yourfile.crt
```


# Trust Root Store
## RedHat Enterprise Linux
1. Copy the certificate files to `/etc/pki/ca-trust/anchor/sources`
2. Run command `sudo update-ca-trust extract`

## Ubuntu Based Linux
1. Copy the certificate files to `/etc/ssl/certs` & `/usr/local/share/ca-certificates` 
2. Run command `sudo update-ca-certificates`

!!!info "Reference"
    - [Install a root CA certificate in the trust store | Ubuntu](https://ubuntu.com/server/docs/install-a-root-ca-certificate-in-the-trust-store)

