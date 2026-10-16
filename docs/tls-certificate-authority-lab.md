# TLS Certificate Authority Lab

## 1. Objective

The objective of this lab was to replace Apache's default self-signed snakeoil certificate with a server certificate signed by a locally managed Certificate Authority (CA).

The lab was performed in a controlled virtual environment using Linux Mint as the client and Ubuntu Server as the web server.

The exercise covered certificate requests, CA key management, X.509 certificate extensions, certificate signing, Apache configuration, and TLS verification.

## 2. Environment

- Client: Linux Mint
- Server: Ubuntu Server
- Web server: Apache HTTP Server 2.4.66
- Server IP: `192.168.16.129`
- Server private key: `/etc/ssl/private/apache-lab.key`
- Local CA directory: `~/.lab-CA`

The CA private key remained on the Linux Mint client. It was not transferred to the server or committed to the repository.

## 3. Certificate Workflow

### Step 1 — Generate the server private key and CSR

The private key and Certificate Signing Request (CSR) were generated on Ubuntu Server.

```bash
sudo openssl req -new -newkey rsa:3072 -noenc \
  -keyout /etc/ssl/private/apache-lab.key \
  -out /tmp/apache-lab.csr \
  -subj "/C=IT/CN=192.168.16.129" \
  -addext "subjectAltName=IP:192.168.16.129"
```

The private key remained on the server. Only the CSR was transferred to the client for signing.

### Step 2 — Prepare certificate extensions

The extension file on Linux Mint was named `server-ext-cnf`.

```text
basicConstraints=CA:FALSE
keyUsage=critical,digitalSignature,keyEncipherment
extendedKeyUsage=serverAuth
subjectAltName=IP:192.168.16.129
```

These extensions identify the certificate as an end-entity certificate intended for TLS server authentication and bind it to the server IP address.

### Step 3 — Sign the CSR using the local CA

The following command was executed on Linux Mint:

```bash
openssl x509 -req \
  -in ~/.lab-CA/apache-lab.csr \
  -CA ~/.lab-CA/ca-certificate.pem \
  -CAkey ~/.lab-CA/ca-private-key.pem \
  -CAcreateserial \
  -out ~/.lab-CA/apache-lab-cert.pem \
  -days 90 \
  -sha256 \
  -extfile ~/.lab-CA/server-ext-cnf
```

The CA private key was used locally to sign the certificate. It was never copied to Ubuntu Server.

### Step 4 — Verify the signed certificate

```bash
openssl verify \
  -CAfile ~/.lab-CA/ca-certificate.pem \
  -verify_ip 192.168.16.129 \
  ~/.lab-CA/apache-lab-cert.pem
```

The expected result was a successful verification ending in `OK`.

### Step 5 — Transfer and install the server certificate

Only the signed server certificate was transferred from Linux Mint to Ubuntu Server.

```bash
scp ~/.lab-CA/apache-lab-cert.pem massiserver@192.168.16.129:/tmp/
```

On Ubuntu Server:

```bash
sudo install -o root -g root -m 644 \
  /tmp/apache-lab-cert.pem \
  /etc/ssl/certs/apache-lab-cert.pem
```

The server private key remained at `/etc/ssl/private/apache-lab.key`.

### Step 6 — Update Apache TLS configuration

The configuration file was backed up before editing:

```bash
sudo cp /etc/apache2/sites-enabled/default-ssl.conf \
  /etc/apache2/sites-enabled/default-ssl.conf.bak
```

The certificate directives were updated to:

```apache
SSLCertificateFile /etc/ssl/certs/apache-lab-cert.pem
SSLCertificateKeyFile /etc/ssl/private/apache-lab.key
```

The Apache configuration was checked before reloading:

```bash
sudo apache2ctl configtest
```

Expected result:

```text
Syntax OK
```

Apache was then reloaded and its status checked:

```bash
sudo systemctl reload apache2
sudo systemctl status apache2 --no-pager
```

The service reported `active (running)` and the reload completed successfully.

### Step 7 — Verify HTTPS from the client

From Linux Mint:

```bash
curl --cacert ~/.lab-CA/ca-certificate.pem \
  -I https://192.168.16.129
```

The server returned:

```text
HTTP/1.1 200 OK
```

The request succeeded without disabling certificate verification with `-k`.

## 4. Results

- A server CSR was generated on Ubuntu Server.
- A local CA signed the server certificate.
- The server certificate was installed and configured in Apache.
- Apache passed its configuration syntax test and reloaded successfully.
- The client successfully validated the TLS certificate using the local CA.
- HTTPS returned an HTTP 200 response.

## 5. Complexity Assessment and Lessons Learned

**Difficulty: High for the current learning stage.**

This lab required several concepts that are normally learned separately:

- Public-key cryptography and private-key protection.
- Certificate Signing Requests and X.509 certificates.
- Certificate Authority trust and certificate extensions.
- Subject Alternative Names and IP address validation.
- Secure file transfer and Linux file permissions.
- Apache TLS configuration and service reloads.
- OpenSSL command-line options and certificate verification.

The exercise was completed with guided assistance. Successful execution does not yet demonstrate independent mastery of the full certificate lifecycle.

The main learning gap identified was the number of unfamiliar concepts introduced at the same time. Future labs should introduce prerequisites in smaller steps, verify understanding before advancing, and distinguish between following instructions and independently diagnosing a problem.

## 6. Security Considerations and Limitations

- The CA private key must remain protected and must never be committed to Git.
- The server private key must remain protected on Ubuntu Server.
- The CA certificate is public, but clients must explicitly trust it or install it in their trusted CA store.
- The certificate's IP address must match the address used by the client.
- The successful `curl --cacert` test confirms that this client trusted the supplied CA for this request. It does not establish automatic trust by every browser or operating system.
- This is a learning lab, not a complete production PKI deployment.

## 7. Evidence

The following checks were successful:

- OpenSSL certificate verification using the local CA.
- Apache configuration test: `Syntax OK`.
- Apache service status: `active (running)`.
- HTTPS request using `curl --cacert`: `HTTP/1.1 200 OK`.

## 8. Next Steps

1. Document the final Apache configuration and certificate workflow.
2. Practise explaining the purpose of the CSR, CA certificate, CA private key, and server private key.
3. Learn how to inspect certificate details and expiration dates.
4. Verify the server certificate using OpenSSL and TLS client tools.
5. Continue with Apache log analysis, security monitoring, and GRC evidence collection.
