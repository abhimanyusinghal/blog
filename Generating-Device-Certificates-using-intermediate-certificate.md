A step-by-step guide on how to generate a device certificate signed by the intermediate certificate. 

### Prerequisites:

- OpenSSL
- The root CA certificate (`rootCA.crt`)
- The intermediate CA certificate (`intermediateCA.crt`)
- The intermediate CA private key (`intermediateCA.key`)

Assuming you've already created the root and intermediate certificates and private keys, follow the steps below.

### 1. Setup a directory structure

We will first set up a directory structure to maintain our certificates and keys organized:

```bash
mkdir deviceCerts && cd deviceCerts
mkdir newcerts private csr
touch index.txt
echo 1000 > serial
```

### 2. Create a configuration file

Create a file named `device_cert.cnf` with the following contents:

```bash
[ ca ]
default_ca = CA_default

[ CA_default ]
dir = .
certs = $dir/newcerts
new_certs_dir = $dir/newcerts
database = $dir/index.txt
serial = $dir/serial
RANDFILE = $dir/private/.rand

certificate = $dir/intermediateCA.crt
private_key = $dir/private/intermediateCA.key
default_days = 365
default_md = sha256
preserve = no
email_in_dn = no
nameopt = default_ca
certopt = default_ca
policy = policy_match

[ policy_match ]
countryName = match
stateOrProvinceName = match
organizationName = match
organizationalUnitName = optional
commonName = supplied
emailAddress = optional

[ req ]
default_bits = 2048
default_keyfile = private/device.key.pem
distinguished_name = req_distinguished_name
attributes = req_attributes
x509_extensions = v3_ca

[ req_distinguished_name ]
countryName = Country Name (2 letter code)
countryName_default = US
stateOrProvinceName = State or Province Name (full name)
localityName = Locality Name (eg, city)
organizationName = Organization Name (eg, company)
organizationalUnitName = Organizational Unit Name (eg, section)
commonName = Common Name (eg, fully qualified host name)
emailAddress = Email Address

[ req_attributes ]
challengePassword = A challenge password

[ v3_ca ]
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid:always,issuer
basicConstraints = CA:FALSE
keyUsage = digitalSignature, keyEncipherment
```

### 3. Generate the device private key:

```bash
openssl genpkey -algorithm RSA -out private/device.key.pem
chmod 400 private/device.key.pem
```

### 4. Create a Certificate Signing Request (CSR) for the device:

Make sure to provide appropriate details when prompted:

```bash
openssl req -config device_cert.cnf -key private/device.key.pem -new -sha256 -out csr/device.csr.pem
```

### 5. Sign the CSR with the intermediate certificate:

```bash
openssl ca -config device_cert.cnf -extensions v3_ca -days 365 -notext -md sha256 -in csr/device.csr.pem -out newcerts/device.crt.pem
```

After completing these steps, you'll have:

- A device private key (`private/device.key.pem`)
- A device certificate signed by the intermediate certificate (`newcerts/device.crt.pem`)

Note that the root CA's private key was never required or exposed during this process, ensuring its security.