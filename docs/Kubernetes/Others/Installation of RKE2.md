---
share: true
tags:
created: 2025-10-31
modified: 2026-01-28
---

# Installation of RKE2

## Installation of Private Registry

In this document, Harbor, an open source OCI compliant registry is used.

1. Download the offline release package from their [Github page](https://github.com/goharbor/harbor/releases).
2. Unpack the package with `tar xzvf harbor-offline-installer-version.tgz`
3. Obtain a private and public certificate for HTTPS configuration Create Self Signed SAN Certificate
4. Create a folder to store the certificates, and convert the certificate to `cert` format for Docker to use
```bash
# Convert certificate
openssl x509 -inform PEM -in yourdomain.crt -out yourdomain.cert
# Copy certificates to folder
sudo mkdir -p /data/cert
cp yourdomain.crt /data/cert
cp yourdomain.key /data/cert
# Copy certificates to docker
sudo mkdir -p /etc/docker/certs.d/yourdomain.com
sudo cp yourdomain.cert /etc/docker/certs.d/yourdomain.com
sudo cp yourdomain.key /etc/docker/certs.d/yourdomain.com
```
5. Restart Docker Engine `sudo systemctl restart docker`
6. Configure Harbor before installation
```bash
# Change directory to where Harbor is unpacked
cd harbor
cp harbor.yml.tpl harbor.yml
# Update the config file to point to the path of the ssl certificates in /data/cert, as well as the domain name.
vi harbor.yml
```
7. Start installation
```bash
./prepare
docker compose up -d
```


>[!info] Reference
>- [Harbor docs \| Configure HTTPS Access to Harbor](https://goharbor.io/docs/1.10/install-config/configure-https/)
>- [Harbor docs \| Configure the Harbor YML File](https://goharbor.io/docs/1.10/install-config/configure-yml-file/)


## Load RKE2 images to registry

1. Download the image archive from [Github release page](https://github.com/rancher/rke2/releases). The file name should be something like `rke2-images.linux-amd64.tar.zst`
2. Load the docker images `docker load < rke2-images.linux-amd64.tar.zst`
3. Tag and push the loaded images to the registry
```bash
docker login <registry_url> -u <username>
docker tag <tag> <registry tag>
docker push <registry tag>
```