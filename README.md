
# Docker Private Registry — Harbor (HTTPS) — With Trivy Image Scanner Setup

👉 [https://harishnshetty.github.io/projects.html](https://harishnshetty.github.io/projects.html)

[![Video Tutorial](https://github.com/harishnshetty/image-data-project/blob/ca3ebcefe72fa095427da3861d511352e22d8ca6/haribor0.jpg)](https://youtu.be/3BRaMSF2OUE)



---

This README provides the complete setup to install **Harbor with HTTPS** using a **self-signed CA** and **server certificate**, including **Trivy image scanner**.

---

### 🔧 System Requirements

| Resource | Minimum | Recommended |
|-----------|----------|--------------|
| CPU | 2 CPU | 4 CPU |
| Memory | 4 GB | 8 GB |
| Disk | 40 GB | 160 GB |

---

### 🌐 Network Ports

| Port | Protocol | Description |
|------|-----------|-------------|
| 443 | HTTPS | Harbor portal and core API accept HTTPS requests |
| 80 | HTTP | Harbor portal and core API accept HTTP requests |

---

### 🧰 Prerequisites

- Ubuntu / Debian (commands use `apt` and `systemctl`)
- Docker, Docker Compose v2
- openssl, wget, tar, gpg (optional for verification)

**Official Links:**

- [Harbor Website](https://goharbor.io/)
- [Installation Docs](https://goharbor.io/docs/2.14.0/install-config/download-installer/)
- [Harbor GitHub Releases](https://github.com/goharbor/harbor/releases)

---

## 🧱 Install Required Tools

```bash
sudo apt update
sudo apt install -y openssl net-tools wget tar gnupg nano
````

```bash
sudo su -
hostnamectl set-hostname harbor-node1.com

echo "10.220.133.251 harbor-node1.com harbor-node1" >> /etc/hosts
cat /etc/hosts

shutdown -r now
```

---

## 🐳 Docker Installation

Official docs: [https://docs.docker.com/engine/install/ubuntu/](https://docs.docker.com/engine/install/ubuntu/)

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

# Add user to docker group
sudo usermod -aG docker $USER
newgrp docker
docker ps
```

---

## 📦 Download Harbor Installer

**Latest Release:**
[https://goharbor.io/docs/2.14.0/install-config/download-installer/](https://goharbor.io/docs/2.14.0/install-config/download-installer/)

```bash
wget https://github.com/goharbor/harbor/releases/download/v2.14.0/harbor-offline-installer-v2.14.0.tgz
tar xzvf harbor-offline-installer-v2.14.0.tgz
cd harbor
```

---

## 🔐 Create CA and Server Certificates

```bash
mkdir -p data/cert
cd data/cert
```

### Generate a CA certificate private key

```bash
openssl genrsa -out ca.key 4096

openssl req -x509 -new -nodes -sha512 -days 3650 \
  -subj "/C=IN/ST=Karnataka/L=Bangalore/O=Harish N Shetty /OU=DevSecOps/CN=Harbor Root CA" \
  -key ca.key -out ca.crt
```

### Generate server key & CSR

```bash
openssl genrsa -out harbor-node1.com.key 4096

openssl req -sha512 -new -subj "/C=IN/ST=Karnataka/L=Bangalore/O=Harish N Shetty /OU=DevSecOps/CN=harbor-node1.com" \
  -key harbor-node1.com.key -out harbor-node1.com.csr
```

### Create `v3.ext` file

```bash
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
```

### Sign CSR with CA

```bash
openssl x509 -req -sha512 -days 3650 -extfile v3.ext \
  -CA ca.crt -CAkey ca.key -CAcreateserial \
  -in harbor-node1.com.csr -out harbor-node1.com.crt
```

### Optional: Create `.cert` for Docker

```bash
openssl x509 -inform PEM -in harbor-node1.com.crt -out harbor-node1.com.cert
```

### Copy certs to Docker path

```bash
sudo mkdir -p /etc/docker/certs.d/harbor-node1.com
sudo cp harbor-node1.com.cert /etc/docker/certs.d/harbor-node1.com/ca.crt
sudo cp ca.crt /etc/docker/certs.d/harbor-node1.com/ca.crt

sudo systemctl restart docker
cd ../..
```

---

## ⚙️ Configure Harbor

```bash
cp harbor.yml.tmpl harbor.yml
```

Edit `harbor.yml` and set the following:

```yaml
hostname: harbor-node1.com

http:
  port: 80

https:
  port: 443
  certificate: /home/harish/harbor/data/cert/harbor-node1.com.crt
  private_key: /home/harish/harbor/data/cert/harbor-node1.com.key

harbor_admin_password: YourStrongPassword123
```

---

## 🚀 Install Harbor

```bash
./prepare
sudo ./install.sh
```

---

## 🧩 Manage Docker Compose

```bash
docker compose down -v
docker compose up -d
sudo chown -R $USER:docker /home/harish/harbor

docker login harbor-node1.com
```

---

## 🧠 Enable Trivy Image Scanning

[![Video Tutorial](https://github.com/harishnshetty/image-data-project/blob/ca3ebcefe72fa095427da3861d511352e22d8ca6/harbor1.jpg)](https://youtu.be/3BRaMSF2OUE)

```bash
docker compose down 
sudo ./install.sh --with-trivy
sudo chown -R $USER:docker /home/harish/harbor
docker compose up -d
```

---

## 🧾 Notes & Troubleshooting

* Ensure `/etc/hosts` contains:

  ```
  harbor-node1.com → <server IP>
  ```
* Docker clients must trust the CA:
  Copy `ca.crt` to `/etc/docker/certs.d/harbor-node1.com/ca.crt`
* Harbor uses `.crt` and `.key` in paths specified in `harbor.yml`
* For production: use trusted CA (e.g., Let's Encrypt or enterprise CA)
* Verify installer (optional):

  ```bash
  gpg --keyserver hkps://keyserver.ubuntu.com --recv-keys 644FF454C0B4115C
  gpg --verify harbor-offline-installer-*.tgz.asc harbor-offline-installer-*.tgz
  ```

---

✅ **Setup Complete — Harbor (HTTPS) + Trivy Ready!**


## For More Projects
[https://harishnshetty.github.io/projects.html](https://harishnshetty.github.io/projects.html)