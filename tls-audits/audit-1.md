# TLS and Certificate Security Audit

## Overview

This audit compares the TLS and certificate configurations of several public websites. The goal is to identify differences in certificate trust, supported protocol versions, cipher configuration, and overall TLS security posture.

Testing was performed using SSL Labs and OpenSSL. Only passive TLS inspection was performed.

---

## Site 1: tls-v1-1.badssl.com

**SSL Labs Grade:** B

### Certificate

**Status:** Valid and trusted. The certificate is currently within its validity period, is not revoked, and SSL Labs reported no certificate chain issues.

### Protocols

**Enabled:** TLS 1.2, TLS 1.1, TLS 1.0

**Deprecated protocols present:** Yes. TLS 1.1 and TLS 1.0 are still enabled. TLS 1.3 is not supported.

### Ciphers

**Weak ciphers present:** Yes. SSL Labs identified multiple weak cipher suites, including CBC-based cipher suites, RSA key-exchange suites, and 3DES.

### Command-Line Confirmation

OpenSSL successfully verified the certificate chain and completed the handshake using TLS 1.2 with the `ECDHE-RSA-AES128-GCM-SHA256` cipher suite. The server certificate uses a 2048-bit RSA key, and OpenSSL returned `Verify return code: 0 (ok)`.

### Severity

**Rating:** Medium

**What an attacker could actually do because of this:**  
Supporting deprecated TLS versions and weaker legacy cipher suites increases the chance that a client could connect using older cryptographic protections. This expands the attack surface and may expose connections to weaknesses associated with outdated TLS versions and cipher configurations.

**Specific fix:**  
Disable TLS 1.0 and TLS 1.1 and require TLS 1.2 or TLS 1.3. Remove legacy cipher suites, including 3DES, unnecessary CBC-based suites, and static RSA key-exchange suites, and prefer modern authenticated encryption such as AES-GCM with forward secrecy.
---

## Site 2: incomplete-chain.badssl.com

**SSL Labs Grade:** B

### Certificate

**Status:** The server certificate itself is valid and trusted, but the certificate chain is incomplete. SSL Labs reported that only one certificate was provided and specifically flagged the chain as incomplete.

### Protocols

**Enabled:** TLS 1.2, TLS 1.1, TLS 1.0

**Deprecated protocols present:** Yes. TLS 1.1 and TLS 1.0 are still enabled. TLS 1.3 is not supported.

### Ciphers

**Weak ciphers present:** Yes. SSL Labs identified multiple weak cipher suites, including CBC-based suites, static RSA key-exchange suites, and 3DES.

### Command-Line Confirmation

OpenSSL confirmed the certificate-chain problem. The handshake completed using TLS 1.2 with the `ECDHE-RSA-AES128-GCM-SHA256` cipher suite, but certificate verification failed with `unable to get local issuer certificate` and `unable to verify the first certificate`. OpenSSL returned `Verify return code: 21`.

### Severity

**Rating:** Medium

**What an attacker could actually do because of this:**  
An incomplete certificate chain can prevent clients from properly validating the server's identity. Some browsers may be able to recover the missing intermediate certificate automatically, but other clients and applications may reject the connection or generate certificate warnings, which can reduce trust and create inconsistent security behavior.

**Specific fix:**  
Configure the web server to send the complete certificate chain, including the required intermediate certificate or certificates along with the server certificate. The server should provide all certificates necessary for the client to build a trusted path back to a trusted root CA.

Additionally, TLS 1.0 and TLS 1.1 should be disabled and legacy cipher suites should be removed in favor of TLS 1.2 or TLS 1.3 with modern authenticated encryption.
---

## Site 3: untrusted-root.badssl.com

**SSL Labs Grade:** T

### Certificate

**Status:** Not trusted. The server certificate is issued by the BadSSL Untrusted Root Certificate Authority, which is not present in standard browser and operating system trust stores. SSL Labs also reported that the certificate chain contains a self-signed root certificate.

### Protocols

**Enabled:** TLS 1.2, TLS 1.1, TLS 1.0

**Deprecated protocols present:** Yes. TLS 1.1 and TLS 1.0 are still enabled. TLS 1.3 is not supported.

### Ciphers

**Weak ciphers present:** Yes. SSL Labs identified multiple weak cipher suites, including CBC-based suites, static RSA key-exchange suites, and 3DES.

### Command-Line Confirmation

OpenSSL confirmed the trust issue. The TLS handshake completed using TLS 1.2 with the `ECDHE-RSA-AES128-GCM-SHA256` cipher suite, but certificate verification failed because the certificate chain contains an untrusted self-signed root. OpenSSL returned `Verify return code: 19 (self-signed certificate in certificate chain)`.

### Severity

**Rating:** High

**What an attacker could actually do because of this:**  
Because the certificate does not chain back to a trusted certificate authority, clients cannot reliably verify the identity of the server. In a real production environment, this could make it easier for users to accept certificate warnings or connect to an impersonated service, weakening the trust model that TLS is supposed to provide.

**Specific fix:**  
Replace the certificate with one issued by a publicly trusted certificate authority and configure the server to provide the correct certificate chain. The untrusted self-signed root should not be used for a public-facing production service.

TLS 1.0 and TLS 1.1 should also be disabled, and legacy cipher suites should be removed in favor of modern TLS 1.2 or TLS 1.3 configurations.
---

## Site 4: github.com

**SSL Labs Grade:** A+

### Certificate

**Status:** Valid and trusted. GitHub uses a publicly trusted Sectigo certificate, and SSL Labs reported no certificate chain issues. The certificate is currently valid and has not been revoked.

### Protocols

**Enabled:** TLS 1.3, TLS 1.2

**Deprecated protocols present:** No. TLS 1.1, TLS 1.0, SSL 3, and SSL 2 are disabled.

### Ciphers

**Weak ciphers present:** Some older TLS 1.2 cipher suites are still identified by SSL Labs as weak, but GitHub strongly supports modern cipher suites such as AES-GCM and ChaCha20-Poly1305 with forward secrecy. TLS 1.3 uses modern authenticated encryption suites.

### Command-Line Confirmation

OpenSSL successfully verified GitHub's certificate chain and negotiated TLS 1.3 using the `TLS_AES_128_GCM_SHA256` cipher suite. The handshake used X25519 for ephemeral key exchange, and OpenSSL returned `Verify return code: 0 (ok)`.

### Additional Security Controls

GitHub has HTTP Strict Transport Security (HSTS) enabled with a long duration and is included in browser HSTS preload lists. This helps ensure browsers connect to the site using HTTPS rather than allowing insecure HTTP connections.

### Severity

**Rating:** Low

**What an attacker could actually do because of this:**  
No significant TLS configuration weakness was identified that would provide an attacker with a practical way to impersonate the server or downgrade the connection under normal conditions. The remaining older TLS 1.2 cipher suites provide some room for further hardening but do not significantly weaken the overall configuration.

**Specific fix:**  
No urgent remediation is required. GitHub could further reduce its attack surface by removing older TLS 1.2 cipher suites marked as weak by SSL Labs while continuing to prioritize TLS 1.3, forward secrecy, and modern authenticated encryption.
---

## Comparison and Key Takeaways

## Comparison and Key Takeaways

The biggest difference between the sites was how well they handled trust, protocol support, and certificate configuration. The BadSSL examples showed how problems like deprecated TLS versions, incomplete certificate chains, and untrusted roots can weaken a site's overall security even when encryption is still being used. The incomplete-chain site showed that having a valid certificate is not enough if the server does not provide the full chain needed for verification. The untrusted-root site was the most serious example because clients could not establish trust in the certificate authority at all. In comparison, GitHub used TLS 1.2 and TLS 1.3, had a trusted certificate chain, and supported modern cipher suites along with HSTS. This showed that strong TLS security depends on more than just enabling HTTPS; the protocols, ciphers, certificate chain, and trust relationship all need to be configured correctly. Overall, the audit showed how small configuration differences can have a major impact on whether a connection is secure and trustworthy.