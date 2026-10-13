# Apache HTTPS/TLS Hardening

## Objective

Document the TLS hardening applied to the Apache HTTPS service, including the configuration changes, technical validation, security impact, and relevance to Governance, Risk, and Compliance (GRC).

## Environment

- Server: Ubuntu Server
- Web server: Apache HTTP Server 2.4.66
- HTTPS port: TCP 443
- Server address: `192.168.16.129`

## Security Configuration

The following directives are configured in Apache's SSL module configuration:

```apache
SSLProtocol all -SSLv2 -SSLv3 -TLSv1 -TLSv1.1
SSLCipherSuite ECDHE+AESGCM:ECDHE+CHACHA20
```

The protocol directive disables obsolete SSL/TLS versions while retaining TLS 1.2 and TLS 1.3.

The cipher-suite directive restricts the configured cipher selection to ECDHE suites using AES-GCM or ChaCha20-based encryption. Actual negotiation also depends on the TLS version and OpenSSL support.

## Validation and Evidence

Apache configuration syntax was checked with:

```bash
sudo apache2ctl configtest
```

**Result:** `Syntax OK`

The Apache service was reloaded to apply the configuration:

```bash
sudo systemctl reload apache2
```

Supported TLS cipher suites were then assessed from the client using:

```bash
nmap -sV --script ssl-enum-ciphers -p 443 192.168.16.129
```

The scan reported three cipher suites for TLS 1.2 and three for TLS 1.3. All reported suites received an `A` grade from Nmap.

The grade applies to the reported cipher suites; it is not a certification of the overall security of the server.

## GRC Relevance

### Governance

Documenting the TLS configuration establishes a traceable technical security baseline.

### Risk Management

Disabling obsolete protocol versions and restricting cipher suites reduces exposure to weak cryptographic configurations.

### Compliance

The configuration and scan results can support technical control assessments. Compliance must be evaluated against the specific requirements of the selected policy, standard, or framework.

### Evidence and Accountability

Configuration validation and client-side scanning provide repeatable evidence for technical reviews and future audits. Before-and-after scan results should be retained when available.

## Cybersecurity Impact and Limitations

TLS hardening strengthens the cryptographic options available to HTTPS connections and reduces the attack surface associated with obsolete protocols and weaker cipher suites.

This is a preventive security control, not proof that the server is free from vulnerabilities. Certificate trust and validity, patch management, application security, access controls, logging, and monitoring require separate assessment.

## Follow-up Actions

- Retain the Nmap scan output as assessment evidence.
- Review certificate validity and client trust.
- Keep Apache and OpenSSL security updates current.
- Repeat the TLS assessment periodically.
- Map the implemented configuration to the applicable security policy or control framework.
