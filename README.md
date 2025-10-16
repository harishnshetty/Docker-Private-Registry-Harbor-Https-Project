...existing code...
# Docker Private Registry — Harbor (HTTPS) — Quickfixed README


[https://harishnshetty.github.io/projects.html](https://harishnshetty.github.io/projects.html)

[![Video Tutorial](https://github.com/harishnshetty/image-data-project/blob/f531e97b49cbc61f8997610e3de92bb3bcaf7c9c/ELk.jpg)](https://youtu.be/G6xeBhUgGBo)


This README is a cleaned, minimal, and corrected set of steps to install Harbor with HTTPS using self-signed CA and server certificate.

Resource	Minimum	Recommended
CPU	2 CPU	4 CPU
Mem	4 GB	8 GB
Disk	40 GB	160 GB


Network ports
Harbor requires that the following ports be open on the target host.

Port	Protocol	Description
443	HTTPS	Harbor portal and core API accept HTTPS requests on this port. You can change this port in the configuration file.
80	HTTP	Harbor portal and core API accept HTTP requests on this port. You can change this port in the configuration file.



Prerequisites
- Ubuntu/Debian (commands use apt and systemctl)
- docker, docker-compose or docker compose v2
- openssl, wget, tar, gpg (optional verify)

Website Link

https://goharbor.io/


https://goharbor.io/docs/2.14.0/install-config/download-installer/


https://github.com/goharbor/harbor/releases

## Install tools

sudo apt update
sudo apt install -y openssl wget tar gnupg nano

sudo su -
hostnamectl set-hostname harbor-node1.com

echo "10.75.1.100 harbor-node1.com elk" >> /etc/hosts
cat /etc/hosts

shutdown -r now

## Docker Installation
 
Official docs: https://docs.docker.com/engine/install/ubuntu/

```bash
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update

sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Add user to docker group (log out / in or newgrp to apply)
sudo usermod -aG docker $USER
newgrp docker
docker ps
```


## Download Harbor installer

wget https://github.com/goharbor/harbor/releases/download/v2.14.0/harbor-offline-installer-v2.14.0.tgz
tar xzvf harbor-offline-installer-v2.14.0.tgz
cd harbor

Create CA and server certificate (example host: harbor-node1.com)

sudo mkdir -p /root/harbor/data/cert
cd /root/harbor/data/cert

# Generate a CA certificate private key.

openssl genrsa -out ca.key 4096


openssl req -x509 -new -nodes -sha512 -days 3650 \
  -subj "/C=IN/ST=Karnataka/L=Bangalore/O=Harish N Shetty /OU=DevSecOps/CN=Harbor Root CA" \
  -key ca.key -out ca.crt

# Generate server key & CSR
openssl genrsa -out harbor-node1.com.key 4096

openssl req -sha512 -new -subj "/C=IN/ST=Karnataka/L=Bangalore/O=Harish N Shetty /OU=DevSecOps/CN=harbor-node1.com" \
  -key harbor-node1.com.key -out harbor-node1.com.csr

# v3 ext file (subjectAltName)
cat > v3.ext <<-EOF
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
extendedKeyUsage = serverAuth
subjectAltName = @alt_names

[alt_names]
DNS.1=harbor-node1.com
DNS.2=harbor-node1
DNS.3=localhost
EOF

## Sign CSR with CA

- Use the v3.ext file to generate a certificate for your Harbor host.

- Replace the yourdomain.com in the CSR and CRT file names with the Harbor host name.


openssl x509 -req -sha512 -days 3650 -extfile v3.ext \
  -CA ca.crt -CAkey ca.key -CAcreateserial \
  -in harbor-node1.com.csr -out harbor-node1.com.crt

## Optional: copy cert in PEM (.cert) form for Docker if needed
openssl x509 -inform PEM -in harbor-node1.com.crt -out harbor-node1.com.cert


# Create Docker cert directory for the registry host
sudo mkdir -p /etc/docker/certs.d/harbor-node1.com

# Copy server cert, key and CA (only cert and CA are required by Docker; private key is for Harbor)
sudo cp harbor-node1.com.cert /etc/docker/certs.d/harbor-node1.com/ca.crt
sudo cp ca.crt /etc/docker/certs.d/harbor-node1.com/ca.crt

# Restart Docker
sudo systemctl restart docker


## Configure Harbor (harbor.yml)
# Edit harbor/harbor.yml (in the installer directory)
# Set hostname and point to the certificate and private key you generated
hostname: harbor-node1.com

http:
  port: 80

https:
  port: 443
  certificate: /root/harbor/data/cert/harbor-node1.com.crt
  private_key: /root/harbor/data/cert/harbor-node1.com.key

harbor_admin_password: YourStrongPassword123


## Install Harbor
# From the harbor installer directory
sudo ./prepare
# To install with default components:
sudo ./install.sh


# Manage compose
docker compose down -v
docker compose up -d

docker login harbor-node1.com


# To include Trivy for IMage Scanning :
sudo ./install.sh --with-trivy

docker compose up -d


...existing code...
## Notes and troubleshooting
- Ensure DNS or /etc/hosts contains harbor-node1.com -> server IP.
- Docker clients must trust the CA: place ca.crt under /etc/docker/certs.d/harbor-node1.com/ca.crt (or add CA to system trust store).
- Harbor needs only the certificate (.crt) and private key (.key) in the paths referenced by harbor.yml.
- For production use, use a certificate signed by a trusted CA (Let's Encrypt or corporate CA).
- To verify downloaded installer (optional):
  gpg --keyserver hkps://keyserver.ubuntu.com --recv-keys 644FF454C0B4115C
  gpg --verify harbor-offline-installer-*.tgz.asc harbor-offline-installer-*.tgz

...existing code...