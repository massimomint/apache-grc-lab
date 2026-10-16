# HTTPS Certificate Validation — Evidence

## Control
TLS certificate validation using a trusted Certificate Authority (CA).

## Test
```bash
curl -I --cacert ~/.lab-CA/ca-certificate.pem https://192.168.16.129
```

## Expected result
The server returns `HTTP/1.1 200 OK` without TLS certificate validation errors.

## Observed result
The HTTPS request returned `HTTP/1.1 200 OK` when the CA certificate was explicitly provided using `--cacert`.

## Evidence
The successful request, together with the absence of certificate validation errors, supports the conclusion that the client successfully validated the server certificate against the specified CA certificate.

## Security objective
Verify the server's identity through certificate validation and help protect against connections to servers presenting certificates that are not trusted by the client.

## Limitation
This test confirms certificate validation for this connection. It does not, by itself, prove that every TLS setting is secure or that the entire service is free of vulnerabilities.
