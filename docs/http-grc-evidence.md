# HTTPS GRC Evidence

## Objective
Collect evidence that the Apache server responds over HTTPS and that the client validates the server certificate using the trusted Certificate Authority (CA).

## Test Procedure
```bash
curl -I --cacert ~/.lab-CA/ca-certificate.pem https://192.168.16.129
```

## Observed Result
The server returned `HTTP/1.1 200 OK` without TLS certificate validation errors.

## Supporting Log Evidence
Apache access logs recorded the following request:

```text
192.168.16.130 - - [16/Oct/2026:12:08:04 +0000] "HEAD / HTTP/1.1" 200 2273
```

## Security Control
TLS certificate validation using a trusted CA.

## Risk Addressed
Reduces the risk of accepting an untrusted server certificate, helping the client verify the server's identity.

## Limitations
The HTTP status code alone does not prove certificate validity. The successful `curl` request with `--cacert` and no certificate validation errors provides the relevant validation evidence. The access log corroborates that Apache processed the request.

This test does not establish that all TLS settings are secure or that the server is free of vulnerabilities.
