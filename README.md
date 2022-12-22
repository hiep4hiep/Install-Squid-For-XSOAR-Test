# Install-Squid-For-XSOAR-Test

# Install Squid binary using this repo https://github.com/diladele/squid-ubuntu

## add diladele apt key
```wget -qO - https://packages.diladele.com/diladele_pub.asc | sudo apt-key add -```

## add new repo
```
echo "deb https://squid57.diladele.com/ubuntu/ focal main" \
    > /etc/apt/sources.list.d/squid57.diladele.com.list
```
## and install
```
apt-get update && apt-get install -y \
    squid-common \
    squid-openssl \
    squidclient \
    libecap3 libecap3-dev
```  
# Verify is the Squid binary was installed with SSL options
```
squid -v
```
![image](https://user-images.githubusercontent.com/41276379/209029336-8803a908-2f45-42df-b13f-af17832b5f51.png)

# Stop squid server
```
systemctl stop squid
```

# Generate CA certificate
```
openssl req -new -newkey rsa:2048 -sha256 -days 365 -nodes -x509 -extensions v3_ca -keyout squid-ca-key.pem -out squid-ca-cert.pem
```
This step will generate a cert file squid-ca-cert.pem and a private key squid-ca-key.pem

# Combine the cert and key into a single pem file
```
cat squid-ca-cert.pem squid-ca-key.pem >> squid-ca-cert-key.pem
```

# Create a directory to store the cert
```
mkdir /etc/squid/certs
mv squid-ca-cert-key.pem /etc/squid/certs/
chown proxy:proxy -R /etc/squid/certs
chmod 700 /etc/squid/certs/squid-ca-cert-key.pem
```

# Generate SSL Database for cert generation
```
mkdir -p /var/lib/squid
/usr/lib/squid/security_file_certgen -c -s /var/lib/squid/ssl_db -M 4MB
chown -R proxy:proxy /var/lib/squid
```

# Edit squid configuration file
```
vi /etc/squid/squid.conf
```

### Search for the line `http_access deny all` and change to `http_access allow all`

### Search for the line `http_port 3128` and change to 
```
http_port 3128 ssl-bump cert=/etc/squid/certs/squid-ca-cert-key.pem generate-host-certificates=on dynamic_cert_mem_cache_size=4MB
```

### And finally, add these lines to the end of the squid.conf file
```
acl step1 at_step SslBump1
ssl_bump peek step1
ssl_bump bump all
sslcrtd_program /usr/lib/squid/security_file_certgen -s /var/lib/squid/ssl_db -M 4MB
sslcrtd_children 5
ssl_bump server-first all
sslproxy_cert_error allow all
```

# Enable and start Squid server
```
systemctl enable squid
systemctl start squid
systemctl status squid
```
![image](https://user-images.githubusercontent.com/41276379/209030615-8044a837-be66-4f7c-815f-50a5f114490e.png)

