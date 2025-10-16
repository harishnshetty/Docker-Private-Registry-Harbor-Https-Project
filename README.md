# Docker-Private-Registry-Harbor-Https-Project


sudo apt update
sudo apt install openssl -y


wget https://github.com/goharbor/harbor/releases/download/v2.14.0/harbor-offline-installer-v2.14.0.tgz

tar xzvf harbor-offline-installer-v2.14.0.tgz

~/harbor/data# mkdir cert

openssl genrsa -out ca.key 4096
openssl req -x509 -new -nodes -sha512 -days 3650 \
 -subj "/C=IN/ST=Karnataka/L=Bangalore/O=HNS/OU=DevSecops/CN=Harbor Root CA" \
 -key ca.key \
 -out ca.crt

openssl genrsa -out harbor-node1.com.key 4096

openssl req -sha512 -new \
  -subj "/C=IN/ST=Karnataka/L=Bangalore/O=HNS/OU=DevSecOps/CN=harbor-node1.com" \
  -key harbor-node1.com.key \
  -out harbor-node1.com.csr

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


openssl x509 -req -sha512 -days 3650 \
  -extfile v3.ext \
  -CA ca.crt -CAkey ca.key -CAcreateserial \
  -in harbor-node1.com.csr \
  -out harbor-node1.com.crt


openssl x509 -inform PEM -in harbor-node1.com.crt -out harbor-node1.com.cert 


sudo mkdir -p /etc/docker/certs.d/harbor-node1.com/

sudo cp harbor-node1.com.cert /etc/docker/certs.d/harbor-node1.com/
sudo cp harbor-node1.com.key /etc/docker/certs.d/harbor-node1.com/
sudo cp ca.crt /etc/docker/certs.d/harbor-node1.com/

systemctl restart docker

sudo vim /root/harbor/harbor.yml
  certificate: /root/harbor/data/cert/harbor-node1.com.crt
  private_key: /root/harbor/data/cert/harbor-node1.com.key

hostname: harbor-node1.com
  
# http related config
http:
  port: 80

# https related config
https:
  port: 443
  certificate: /root/harbor/data/cert/harbor-node1.com.crt
  private_key: /root/harbor/data/cert/harbor-node1.com.key



./prepare

./install
docker compose down -v

docker compose up -d


sudo ./install.sh --with-trivy







https://github.com/goharbor/harbor/releases
https://goharbor.io/docs/2.14.0/install-config/download-installer/
https://github.com/goharbor/harbor/releases/download/v2.14.0/harbor-offline-installer-v2.14.0.tgz


gpg --keyserver hkps://keyserver.ubuntu.com --receive-keys 644FF454C0B4115C

wget https://github.com/goharbor/harbor/releases/download/v2.14.0/harbor-online-installer-v2.14.0.tgz



gpg -v --keyserver hkps://keyserver.ubuntu.com --verify harbor-online-installer-*.tgz.asc


tar xzvf harbor-online-installer-version.tgz

cd harbor

cp harbor.yml.tmpl harbor.yml
vim harbor.yml

hostname: harbor.yourdomain.com

https:
  port: 443
  certificate: /etc/harbor/certs/harbor.crt
  private_key: /etc/harbor/certs/harbor.key

harbor_admin_password: YourStrongPassword123

https configuration 



sudo mkdir -p /etc/harbor/certs
cd /etc/harbor/certs

openssl genrsa -out harish.key 4096

openssl req -sha512 -new \
    -subj "/C=CN/ST=Beijing/L=Beijing/O=example/OU=Personal/CN=harish" \
    -key harish.key \
    -out harish.csr

    cat > v3.ext <<-EOF
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
extendedKeyUsage = serverAuth
subjectAltName = @alt_names

[alt_names]
DNS.1=yourdomain.com
DNS.2=yourdomain
DNS.3=hostname
EOF

openssl x509 -req -sha512 -days 3650 \
    -extfile v3.ext \
    -CA ca.crt -CAkey ca.key -CAcreateserial \
    -in yourdomain.com.csr \
    -out yourdomain.com.crt

    cp yourdomain.com.crt /data/cert/
cp yourdomain.com.key /data/cert/

openssl x509 -inform PEM -in yourdomain.com.crt -out yourdomain.com.cert


cp yourdomain.com.cert /etc/docker/certs.d/yourdomain.com/
cp yourdomain.com.key /etc/docker/certs.d/yourdomain.com/
cp ca.crt /etc/docker/certs.d/yourdomain.com/

systemctl restart docker

./prepare

docker compose down -v
docker compose up -d

docker login yourdomain.com

docker login yourdomain.com:port


 sudo mkdir -p /etc/harbor/certs
cd /etc/harbor/certs
sudo openssl req -newkey rsa:4096 -nodes -sha256 -keyout harbor.key -x509 -days 365 -out harbor.crt
