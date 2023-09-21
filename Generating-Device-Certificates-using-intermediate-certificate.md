A step-by-step guide on how to generate a device certificate signed by the intermediate certificate. 

### Prerequisites:

- OpenSSL
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
countryName = optional
stateOrProvinceName = optional
organizationName = optional
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


The `.cnf` configuration file for OpenSSL contains settings used during various certificate-related operations. Below is break down the contents of the file above:

### [ca]

This section defines the default CA commands. 

- `default_ca`: This specifies the name of the default section to use when the `ca` command is run. 

### [CA_default]

This section specifies default settings for the CA utility:

- `dir`: The root directory for the certificate and key files.
- `certs`: Where to find existing certificates.
- `new_certs_dir`: Where to place new certificates.
- `database`: The text database file that keeps track of signed certificates.
- `serial`: The file that holds the current serial number.
- `RANDFILE`: Location of the random number file.
- `certificate`: The location of the intermediate certificate.
- `private_key`: The location of the private key for the intermediate CA.
- `default_days`: How many days a certificate is valid for by default.
- `default_md`: The default message digest to use.
- `preserve`: If set to "yes", it preserves the existing files by renaming them with a `.bak` extension.
- `email_in_dn`: Whether to include an email address in the distinguished name.
- `nameopt` and `certopt`: These are naming options and certificate display options.
- `policy`: Policy to use when validating certificate signing requests (CSR). Here, it refers to the `policy_match` section.

### [policy_match]

This section defines the expected fields in the certificate signing request. Each field can have values like `match`, `optional`, or `supplied`.

- `match`: The field in the CSR must match the CA's certificate.
- `optional`: The field may be present but can be left blank.
- `supplied`: The field must be present and cannot be left blank.

### [req]

This section is used when creating private keys and certificate signing requests:

- `default_bits`: The size of the RSA key.
- `default_keyfile`: The default name for the private key file.
- `distinguished_name`: Section to use when prompting for fields for the Distinguished Name.
- `attributes`: The attributes to prompt for when generating a CSR.
- `x509_extensions`: Extensions to use when creating a self-signed certificate (not required for CSRs).

### [req_distinguished_name]

This section specifies the fields to prompt for when creating a CSR or self-signed certificate. These are the fields of the distinguished name in the certificate.

### [req_attributes]

This section specifies additional attributes to prompt for when creating a CSR.

### [v3_ca]

This section defines x509 v3 extensions that can be included in the certificate.

- `subjectKeyIdentifier`: Creates an extension with a hash of the subject's public key. This can be used to identify the public key in the certificate.
- `authorityKeyIdentifier`: Creates an extension with a hash of the issuer's public key. This helps identify the issuer's public key.
- `basicConstraints`: Indicates if the certificate is a CA certificate. `CA:FALSE` means it's not a CA certificate.
- `keyUsage`: This specifies the cryptographic operations for which the certificate's public key can be used.
