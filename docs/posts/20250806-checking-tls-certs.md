---
title: "Checking TLS Certificates"
date:
  created: 2025-08-06

linktitle: "Checking TLS Certificates"
slug: "checking-tls-certificates"

description: "How to check TLS certificates with CLI tools"

tags:
- CLI
- SSL
- TLS
- Security
- Webserver

authors:
- harry

draft: false
---
# How to check TLS certificates (and more)

From time to time, I want to check the TLS certificates of different websites. This can be done with several command line tools.

<!-- more -->

## `curl`

```sh
curl -vl https://pixelchrome.org
```

Output:

```sh
* Host pixelchrome.org:443 was resolved.
* IPv6: 2a01:4f8:1c1b:4481::1
* IPv4: 91.107.227.60
*   Trying [2a01:4f8:1c1b:4481::1]:443...
* Connected to pixelchrome.org (2a01:4f8:1c1b:4481::1) port 443
* ALPN: curl offers h2,http/1.1
* (304) (OUT), TLS handshake, Client hello (1):
*  CAfile: /etc/ssl/cert.pem
*  CApath: none
* (304) (IN), TLS handshake, Server hello (2):
* (304) (IN), TLS handshake, Unknown (8):
* (304) (IN), TLS handshake, Certificate (11):
* (304) (IN), TLS handshake, CERT verify (15):
* (304) (IN), TLS handshake, Finished (20):
* (304) (OUT), TLS handshake, Finished (20):
* SSL connection using TLSv1.3 / AEAD-AES256-GCM-SHA384 / [blank] / UNDEF
* ALPN: server accepted h2
* Server certificate:
*  subject: CN=pixelchrome.org
*  start date: Aug  3 00:14:56 2025 GMT
*  expire date: Nov  1 00:14:55 2025 GMT
*  subjectAltName: host "pixelchrome.org" matched cert's "pixelchrome.org"
*  issuer: C=US; O=Let's Encrypt; CN=E5
*  SSL certificate verify ok.
* using HTTP/2
* [HTTP/2] [1] OPENED stream for https://pixelchrome.org/
* [HTTP/2] [1] [:method: GET]
* [HTTP/2] [1] [:scheme: https]
* [HTTP/2] [1] [:authority: pixelchrome.org]
* [HTTP/2] [1] [:path: /]
* [HTTP/2] [1] [user-agent: curl/8.7.1]
* [HTTP/2] [1] [accept: */*]
> GET / HTTP/2
> Host: pixelchrome.org
> User-Agent: curl/8.7.1
> Accept: */*
>
* Request completely sent off
< HTTP/2 301
< server: nginx
< date: Wed, 06 Aug 2025 16:31:55 GMT
< content-type: text/html; charset=UTF-8
< location: https://pixelchrome.org/blog
<
* Connection #0 to host pixelchrome.org left intact
```

## `nmap`

```sh
nmap -p 443 --script ssl-cert pixelchrome.org
```

Output:

```sh
Starting Nmap 7.93 ( https://nmap.org ) at 2025-08-06 18:36 CEST
Nmap scan report for pixelchrome.org (91.107.227.60)
Host is up (0.0088s latency).
Other addresses for pixelchrome.org (not scanned): 2a01:4f8:1c1b:4481::1
rDNS record for 91.107.227.60: static.60.227.107.91.clients.your-server.de

PORT    STATE SERVICE
443/tcp open  https
| ssl-cert: Subject: commonName=pixelchrome.org
| Subject Alternative Name: DNS:pixelchrome.org
| Issuer: commonName=E5/organizationName=Let's Encrypt/countryName=US
| Public Key type: ec
| Public Key bits: 384
| Signature Algorithm: ecdsa-with-SHA384
| Not valid before: 2025-08-03T00:14:56
| Not valid after:  2025-11-01T00:14:55
| MD5:   f7deb2ad30df86423fd353ea008e6bad
|_SHA-1: 5ed01a45dd2ef3aed20216da74d1a4f4b45e18a9

Nmap done: 1 IP address (1 host up) scanned in 0.36 seconds
```

### check with `nmap` which TLS versions and ciphers are supported

```sh
nmap --script ssl-enum-ciphers -p 443 pixelchrome.org
```

Output:

```sh
Starting Nmap 7.93 ( https://nmap.org ) at 2025-08-06 19:09 CEST
Nmap scan report for pixelchrome.org (91.107.227.60)
Host is up (0.00052s latency).
Other addresses for pixelchrome.org (not scanned): 2a01:4f8:1c1b:4481::1
rDNS record for 91.107.227.60: static.60.227.107.91.clients.your-server.de

PORT    STATE SERVICE
443/tcp open  https
| ssl-enum-ciphers:
|   TLSv1.2:
|     ciphers:
|       TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256 (secp256r1) - A
|       TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305_SHA256 (secp256r1) - A
|       TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384 (secp256r1) - A
|       TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA (secp256r1) - A
|       TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA (secp256r1) - A
|     compressors:
|       NULL
|     cipher preference: server
|     warnings:
|       Key exchange (secp256r1) of lower strength than certificate key
|   TLSv1.3:
|     ciphers:
|       TLS_AKE_WITH_AES_128_GCM_SHA256 (ecdh_x25519) - A
|       TLS_AKE_WITH_AES_256_GCM_SHA384 (ecdh_x25519) - A
|       TLS_AKE_WITH_CHACHA20_POLY1305_SHA256 (ecdh_x25519) - A
|     cipher preference: server
|_  least strength: A

Nmap done: 1 IP address (1 host up) scanned in 0.42 seconds
```

## `openssl`

```sh
echo | openssl s_client -showcerts -connect pixelchrome.org:443 2>/dev/null | openssl x509 -text
```

Output:

```sh
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            05:f6:15:6b:eb:50:ef:a0:fc:6a:f8:48:37:84:b9:27:bd:2d
        Signature Algorithm: ecdsa-with-SHA384
        Issuer: C = US, O = Let's Encrypt, CN = E5
        Validity
            Not Before: Aug  3 00:14:56 2025 GMT
            Not After : Nov  1 00:14:55 2025 GMT
        Subject: CN = pixelchrome.org
        Subject Public Key Info:
            Public Key Algorithm: id-ecPublicKey
                Public-Key: (384 bit)
                pub:
                    04:46:0f:64:4e:fd:9f:96:0f:f0:75:56:a6:06:1d:
                    99:09:be:c5:64:28:be:ae:ca:9b:6a:cd:82:30:cb:
                    b4:22:a9:87:33:de:82:37:1e:c6:36:26:17:be:b1:
                    d4:f4:9f:c1:27:3e:21:1c:ac:1e:77:c6:06:3b:f4:
                    bc:51:18:4c:e5:df:fe:d6:4c:06:1d:bf:0d:f9:f9:
                    69:ed:bf:ec:57:f6:b8:c9:f6:2c:ee:38:f5:41:4d:
                    ac:1c:b2:85:0c:64:e1
                ASN1 OID: secp384r1
                NIST CURVE: P-384
        X509v3 extensions:
            X509v3 Key Usage: critical
                Digital Signature
            X509v3 Extended Key Usage:
                TLS Web Server Authentication, TLS Web Client Authentication
            X509v3 Basic Constraints: critical
                CA:FALSE
            X509v3 Subject Key Identifier:
                EA:4A:22:5F:7A:55:8B:45:EB:2F:E5:74:B9:BE:4C:DA:26:53:87:E0
            X509v3 Authority Key Identifier:
                9F:2B:5F:CF:3C:21:4F:9D:04:B7:ED:2B:2C:C4:C6:70:8B:D2:D7:0D
            Authority Information Access:
                CA Issuers - URI:http://e5.i.lencr.org/
            X509v3 Subject Alternative Name:
                DNS:pixelchrome.org
            X509v3 Certificate Policies:
                Policy: 2.23.140.1.2.1
            X509v3 CRL Distribution Points:
                Full Name:
                  URI:http://e5.c.lencr.org/46.crl
            CT Precertificate SCTs:
                Signed Certificate Timestamp:
                    Version   : v1 (0x0)
                    Log ID    : ED:3C:4B:D6:E8:06:C2:A4:A2:00:57:DB:CB:24:E2:38:
                                01:DF:51:2F:ED:C4:86:C5:70:0F:20:DD:B7:3E:3F:E0
                    Timestamp : Aug  3 01:13:26.875 2025 GMT
                    Extensions: none
                    Signature : ecdsa-with-SHA256
                                30:46:02:21:00:BC:98:92:FA:25:39:D2:06:87:C6:2B:
                                F5:C9:AB:50:A7:48:8C:88:22:67:E7:B6:17:9B:23:56:
                                AB:20:B0:DA:E3:02:21:00:C6:79:41:C2:31:21:59:7B:
                                83:EC:89:F4:A7:A1:FB:D9:C3:EE:CA:1A:FD:E3:37:70:
                                20:F1:AC:FB:1D:F1:8C:8D
                Signed Certificate Timestamp:
                    Version   : v1 (0x0)
                    Log ID    : CC:FB:0F:6A:85:71:09:65:FE:95:9B:53:CE:E9:B2:7C:
                                22:E9:85:5C:0D:97:8D:B6:A9:7E:54:C0:FE:4C:0D:B0
                    Timestamp : Aug  3 01:13:26.890 2025 GMT
                    Extensions: none
                    Signature : ecdsa-with-SHA256
                                30:44:02:20:16:44:4D:D4:D5:B2:75:B1:60:67:6E:DF:
                                D8:C9:56:A1:73:36:EE:67:D8:7D:37:14:61:45:A8:39:
                                F3:0A:9D:EB:02:20:1B:5F:55:E2:F2:9F:33:15:8B:B6:
                                0B:BB:C7:43:09:A8:C1:AB:A4:74:BF:B7:5B:13:8B:6D:
                                B1:F6:4F:30:52:D1
    Signature Algorithm: ecdsa-with-SHA384
    Signature Value:
        30:65:02:30:1a:d5:9c:62:3e:0b:d0:70:a1:f2:80:0d:33:e4:
        32:43:63:06:9f:82:ba:b6:e5:92:04:75:3a:46:6f:67:83:43:
        be:8e:19:98:67:5f:f6:f7:2f:b4:cc:9d:e6:0b:4c:c8:02:31:
        00:e8:1b:45:1e:ff:72:34:d8:a0:db:c0:4f:80:e2:05:6c:ea:
        f5:ba:d1:7c:e3:25:4c:4b:0c:99:0f:84:cf:7f:46:a2:0b:5c:
        d4:09:22:17:7e:26:6b:a8:e9:4c:ba:39:b6
-----BEGIN CERTIFICATE-----
MIIDqDCCAy6gAwIBAgISBfYVa+tQ76D8avhIN4S5J70tMAoGCCqGSM49BAMDMDIx
CzAJBgNVBAYTAlVTMRYwFAYDVQQKEw1MZXQncyBFbmNyeXB0MQswCQYDVQQDEwJF
NTAeFw0yNTA4MDMwMDE0NTZaFw0yNTExMDEwMDE0NTVaMBoxGDAWBgNVBAMTD3Bp
eGVsY2hyb21lLm9yZzB2MBAGByqGSM49AgEGBSuBBAAiA2IABEYPZE79n5YP8HVW
pgYdmQm+xWQovq7Km2rNgjDLtCKphzPegjcexjYmF76x1PSfwSc+IRysHnfGBjv0
vFEYTOXf/tZMBh2/Dfn5ae2/7Ff2uMn2LO449UFNrByyhQxk4aOCAh0wggIZMA4G
A1UdDwEB/wQEAwIHgDAdBgNVHSUEFjAUBggrBgEFBQcDAQYIKwYBBQUHAwIwDAYD
VR0TAQH/BAIwADAdBgNVHQ4EFgQU6koiX3pVi0XrL+V0ub5M2iZTh+AwHwYDVR0j
BBgwFoAUnytfzzwhT50Et+0rLMTGcIvS1w0wMgYIKwYBBQUHAQEEJjAkMCIGCCsG
AQUFBzAChhZodHRwOi8vZTUuaS5sZW5jci5vcmcvMBoGA1UdEQQTMBGCD3BpeGVs
Y2hyb21lLm9yZzATBgNVHSAEDDAKMAgGBmeBDAECATAtBgNVHR8EJjAkMCKgIKAe
hhxodHRwOi8vZTUuYy5sZW5jci5vcmcvNDYuY3JsMIIBBAYKKwYBBAHWeQIEAgSB
9QSB8gDwAHcA7TxL1ugGwqSiAFfbyyTiOAHfUS/txIbFcA8g3bc+P+AAAAGYbX2i
WwAABAMASDBGAiEAvJiS+iU50gaHxiv1yatQp0iMiCJn57YXmyNWqyCw2uMCIQDG
eUHCMSFZe4PsifSnofvZw+7KGv3jN3Ag8az7HfGMjQB1AMz7D2qFcQll/pWbU87p
snwi6YVcDZeNtql+VMD+TA2wAAABmG19omoAAAQDAEYwRAIgFkRN1NWydbFgZ27f
2MlWoXM27mfYfTcUYUWoOfMKnesCIBtfVeLynzMVi7YLu8dDCajBq6R0v7dbE4tt
sfZPMFLRMAoGCCqGSM49BAMDA2gAMGUCMBrVnGI+C9BwofKADTPkMkNjBp+Curbl
kgR1OkZvZ4NDvo4ZmGdf9vcvtMyd5gtMyAIxAOgbRR7/cjTYoNvAT4DiBWzq9brR
fOMlTEsMmQ+Ez39Gogtc1AkiF34ma6jpTLo5tg==
-----END CERTIFICATE-----
```

### get even more details with `openssl`

```sh
openssl s_client -connect pixelchrome.org:443
```

Check the `SSL-Session: Protocol:` section to find out more about the TLS version used.

Output:

```sh hl_lines="64-65"
CONNECTED(00000003)
depth=2 C = US, O = Internet Security Research Group, CN = ISRG Root X1
verify return:1
depth=1 C = US, O = Let's Encrypt, CN = E5
verify return:1
depth=0 CN = pixelchrome.org
verify return:1
---
Certificate chain
 0 s:CN = pixelchrome.org
   i:C = US, O = Let's Encrypt, CN = E5
   a:PKEY: id-ecPublicKey, 384 (bit); sigalg: ecdsa-with-SHA384
   v:NotBefore: Aug  3 00:14:56 2025 GMT; NotAfter: Nov  1 00:14:55 2025 GMT
 1 s:C = US, O = Let's Encrypt, CN = E5
   i:C = US, O = Internet Security Research Group, CN = ISRG Root X1
   a:PKEY: id-ecPublicKey, 384 (bit); sigalg: RSA-SHA256
   v:NotBefore: Mar 13 00:00:00 2024 GMT; NotAfter: Mar 12 23:59:59 2027 GMT
---
Server certificate
-----BEGIN CERTIFICATE-----
MIIDqDCCAy6gAwIBAgISBfYVa+tQ76D8avhIN4S5J70tMAoGCCqGSM49BAMDMDIx
CzAJBgNVBAYTAlVTMRYwFAYDVQQKEw1MZXQncyBFbmNyeXB0MQswCQYDVQQDEwJF
NTAeFw0yNTA4MDMwMDE0NTZaFw0yNTExMDEwMDE0NTVaMBoxGDAWBgNVBAMTD3Bp
eGVsY2hyb21lLm9yZzB2MBAGByqGSM49AgEGBSuBBAAiA2IABEYPZE79n5YP8HVW
pgYdmQm+xWQovq7Km2rNgjDLtCKphzPegjcexjYmF76x1PSfwSc+IRysHnfGBjv0
vFEYTOXf/tZMBh2/Dfn5ae2/7Ff2uMn2LO449UFNrByyhQxk4aOCAh0wggIZMA4G
A1UdDwEB/wQEAwIHgDAdBgNVHSUEFjAUBggrBgEFBQcDAQYIKwYBBQUHAwIwDAYD
VR0TAQH/BAIwADAdBgNVHQ4EFgQU6koiX3pVi0XrL+V0ub5M2iZTh+AwHwYDVR0j
BBgwFoAUnytfzzwhT50Et+0rLMTGcIvS1w0wMgYIKwYBBQUHAQEEJjAkMCIGCCsG
AQUFBzAChhZodHRwOi8vZTUuaS5sZW5jci5vcmcvMBoGA1UdEQQTMBGCD3BpeGVs
Y2hyb21lLm9yZzATBgNVHSAEDDAKMAgGBmeBDAECATAtBgNVHR8EJjAkMCKgIKAe
hhxodHRwOi8vZTUuYy5sZW5jci5vcmcvNDYuY3JsMIIBBAYKKwYBBAHWeQIEAgSB
9QSB8gDwAHcA7TxL1ugGwqSiAFfbyyTiOAHfUS/txIbFcA8g3bc+P+AAAAGYbX2i
WwAABAMASDBGAiEAvJiS+iU50gaHxiv1yatQp0iMiCJn57YXmyNWqyCw2uMCIQDG
eUHCMSFZe4PsifSnofvZw+7KGv3jN3Ag8az7HfGMjQB1AMz7D2qFcQll/pWbU87p
snwi6YVcDZeNtql+VMD+TA2wAAABmG19omoAAAQDAEYwRAIgFkRN1NWydbFgZ27f
2MlWoXM27mfYfTcUYUWoOfMKnesCIBtfVeLynzMVi7YLu8dDCajBq6R0v7dbE4tt
sfZPMFLRMAoGCCqGSM49BAMDA2gAMGUCMBrVnGI+C9BwofKADTPkMkNjBp+Curbl
kgR1OkZvZ4NDvo4ZmGdf9vcvtMyd5gtMyAIxAOgbRR7/cjTYoNvAT4DiBWzq9brR
fOMlTEsMmQ+Ez39Gogtc1AkiF34ma6jpTLo5tg==
-----END CERTIFICATE-----
subject=CN = pixelchrome.org
issuer=C = US, O = Let's Encrypt, CN = E5
---
No client certificate CA names sent
Peer signing digest: SHA384
Peer signature type: ECDSA
Server Temp Key: X25519, 253 bits
---
SSL handshake has read 2467 bytes and written 401 bytes
Verification: OK
---
New, TLSv1.3, Cipher is TLS_AES_256_GCM_SHA384
Server public key is 384 bit
Secure Renegotiation IS NOT supported
Compression: NONE
Expansion: NONE
No ALPN negotiated
Early data was not sent
Verify return code: 0 (ok)
---
---
Post-Handshake New Session Ticket arrived:
SSL-Session:
    Protocol  : TLSv1.3
    Cipher    : TLS_AES_256_GCM_SHA384
    Session-ID: 1328CBAAF9D7C84CA0463FA9A33121E74AFD94AA44002AE60A4FC1FEE9F616EE
    Session-ID-ctx:
    Resumption PSK: E2467253E2A31E506B593D1EFF6A3555074B2FE6374E36F81AA697FF27C25D0827ECA93297579FBA970507624DA0BD4E
    PSK identity: None
    PSK identity hint: None
    SRP username: None
    TLS session ticket lifetime hint: 86400 (seconds)
    TLS session ticket:
    0000 - 01 1a 12 71 da 29 ff ff-9c 20 89 10 ea c4 3e 35   ...q.)... ....>5
    0010 - 67 06 2f 72 85 89 d3 01-ab 04 f9 b3 69 2b ee d9   g./r........i+..

    Start Time: 1754499360
    Timeout   : 7200 (sec)
    Verify return code: 0 (ok)
    Extended master secret: no
    Max Early Data: 0
---
read R BLOCK
---
Post-Handshake New Session Ticket arrived:
SSL-Session:
    Protocol  : TLSv1.3
    Cipher    : TLS_AES_256_GCM_SHA384
    Session-ID: 73A9F08DA60B67CF15F19D8CDEE1117A2DCEF0CA2C921249E69EFC1032D1C6AF
    Session-ID-ctx:
    Resumption PSK: A30C5A9D094EB07721785403289FF55447F16E9A5F06373B91C6B59F018A7BC19E0535C7A1BB4EED9A6BAA6B153DA77A
    PSK identity: None
    PSK identity hint: None
    SRP username: None
    TLS session ticket lifetime hint: 86400 (seconds)
    TLS session ticket:
    0000 - 36 62 48 cf ed bd 09 98-9a 48 ac e0 70 a4 67 a5   6bH......H..p.g.
    0010 - ce a3 36 5a d4 e3 70 d8-d9 4e 33 bb 5c dd d6 d9   ..6Z..p..N3.\...

    Start Time: 1754499360
    Timeout   : 7200 (sec)
    Verify return code: 0 (ok)
    Extended master secret: no
    Max Early Data: 0
---
read R BLOCK
closed
```

### Test for specific TLS versions

```sh
openssl s_client -connect pixelchrome.org:443 -tls1_3
openssl s_client -connect pixelchrome.org:443 -tls1_2
openssl s_client -connect pixelchrome.org:443 -tls1_1
openssl s_client -connect pixelchrome.org:443 -tls1
```
