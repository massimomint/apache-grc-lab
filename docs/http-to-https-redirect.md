# Apache HTTP to HTTPS Redirect

## Objective

Configure Apache to temporarily redirect HTTP requests to HTTPS and verify the behavior from a client machine.

## Environment

- **Server:** `massiserver`
- **Server IP:** `192.168.16.129`
- **Web server:** Apache HTTP Server 2.4.66
- **Server OS:** Ubuntu 26.04
- **HTTP:** TCP/80
- **HTTPS:** TCP/443

## 1. Configuration Discovery

The HTTP virtual host configuration is located at:

`/etc/apache2/sites-enabled/000-default.conf`

The virtual host listens on port 80 and uses `/var/www/html` as its document root.

Before the change, the server returned an HTTP `200 OK` response and served the website directly over HTTP.

## 2. Redirect Configuration

The following directive was added inside the `<VirtualHost *:80>` block:

```apache
Redirect 302 / https://192.168.16.129
```

This directive redirects HTTP requests to the server's HTTPS endpoint.

The `302 Found` status indicates a temporary redirect. A permanent redirect was not configured.

### Configuration Error and Resolution

An incorrectly formatted destination URL caused an Apache configuration syntax error.

The directive contained an unintended space between `https://` and the server IP address.

After correcting the URL, the configuration test returned `Syntax OK`.

## 3. Configuration Validation and Reload

### Syntax Validation

```bash
sudo apache2ctl configtest
```

Expected result:

`Syntax OK`

### Apache Reload

```bash
sudo systemctl reload apache2
```

Reloading Apache applies the updated configuration without requiring a full service restart.

## 4. Client-Side Verification with curl

### Inspecting the HTTP Response

```bash
curl -i http://192.168.16.129
```

Observed results:

- HTTP status: `302 Found`
- `Location` header: `https://192.168.16.129`

This confirms that the server responds to HTTP requests with a redirect to HTTPS.

### Following the Redirect and Inspecting TLS

```bash
curl -Lv http://192.168.16.129
```

Options used:

- `-L`: follows HTTP redirects.
- `-v`: displays connection and TLS diagnostic information.

The client reached TCP/443 and started the TLS handshake. However, certificate validation failed with a `self-signed certificate` error.

The client does not currently trust the certificate presented by the server.

## 5. Security Assessment

The configuration confirms the HTTP-to-HTTPS redirect behavior, but it does not establish that the entire service is secure.

Important considerations:

- The `302` redirect is temporary.
- The initial HTTP request is not encrypted.
- A self-signed certificate is not automatically trusted by the client.
- Server identity and Subject Alternative Name (SAN) validation must be checked separately.
- A redirect does not replace proper TLS configuration.

## 6. Next Steps

- Create a private Certificate Authority (CA) for the lab.
- Generate the server's private key and Certificate Signing Request (CSR).
- Sign the server certificate using the private CA.
- Configure Apache to use the issued certificate.
- Install the CA's public certificate in the client trust store.
- Verify HTTPS with `curl` without disabling TLS verification.

## Evidence and Limitations

The Apache configuration test, virtual host configuration, and `curl` responses provide evidence of the observed behavior.

Conclusions must be limited to tests actually performed. The redirect does not, by itself, prove certificate trust, complete TLS security, or the absence of vulnerabilities.
