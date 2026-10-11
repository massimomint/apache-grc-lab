# Host Firewall Configuration — UFW

## Objective

Reduce the server's network attack surface by enabling the host-based firewall and allowing the network services required for the lab.

## Environment

* Server: Ubuntu Server (`GRC-Apache-Server`)
* Web service: Apache HTTP Server
* Host firewall: UFW

## Configuration

UFW was enabled with inbound allow rules for:

* TCP/22 — SSH for remote administration
* TCP/80 — HTTP for the Apache web service

IPv4 and IPv6 allow rules are present.

## Verification Evidence

The command `sudo ufw status` reported:

* Firewall status: `active`
* Port 22: `ALLOW`
* Port 80: `ALLOW`

The command `sudo ufw show added` confirmed both configured rules.

## Security Benefit

Enabling UFW establishes a host-based firewall policy for the services required by the lab and reduces exposure to unwanted inbound connections on other ports.

## Residual Risk

Both allowed ports are permitted from `Anywhere`, including IPv4 and IPv6. Source-address restrictions have not yet been configured. Enabling UFW alone does not prove that the server is unreachable from external networks or that the allowed services are free from vulnerabilities.

