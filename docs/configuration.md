## HTTPS/TLS Configuration

### Configuration

Apache uses the `mod_ssl` module to support HTTPS connections secured with TLS.

SSL (Secure Sockets Layer) is an obsolete protocol that has been replaced by TLS (Transport Layer Security). The name `mod_ssl` is historical and does not mean that the obsolete SSL protocol is being used.

TLS provides three main security properties:

* **Confidentiality:** encrypts data transmitted between client and server.
* **Integrity:** helps detect unauthorized modifications to transmitted data.
* **Authentication:** allows the client to verify the server's identity through its digital certificate, provided the certificate is valid and trusted.

The SSL module was enabled in Apache, and the default SSL site was activated. The UFW firewall was configured to allow TCP traffic on port 443, the standard port for HTTPS.

### Verification Evidence

The Apache service was checked and confirmed to be running. The SSL module was verified in Apache's loaded modules, and a listener was observed on TCP port 443.

An HTTPS request to the server returned `HTTP/1.1 200 OK`, confirming that the web server responded successfully over HTTPS.

The server certificate was inspected using OpenSSL. Its subject and issuer were both `CN=massiserver`, and its Subject Alternative Name contained `DNS:massiserver`.

### Security Limitations and Residual Risk

The certificate is self-signed and is not automatically trusted by client systems. The certificate contains the DNS name `massiserver`, but the server IP address was not listed as a Subject Alternative Name.

A successful HTTPS response confirms service availability, but does not prove that the TLS configuration is free of weaknesses or that the web application is secure.

HTTPS protects data in transit, but does not eliminate vulnerabilities in Apache, the application, system configuration, or user credentials. Further TLS configuration checks and security testing are required to assess the overall security posture.
