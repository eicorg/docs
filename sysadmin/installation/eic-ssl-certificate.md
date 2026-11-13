# EIC SSL certificate Generation and Placement

The wildcard belongs inside the CSR’s Common Name and Subject Alternative Name (SAN).    
Putting * in the filenames doesn’t make it a wildcard request—and Windows doesn’t allow * in filenames.

Run this as one line:  
~~~bash
#openssl req -new -newkey rsa:2048 -nodes -sha256 -keyout wildcard.eic.bnl.gov.key -out wildcard.eic.bnl.gov.csr -subj "/CN=*.eic.bnl.gov" -addext "subjectAltName=DNS:*.eic.bnl.gov"
~~~

Here, -subj sets the Common Name, and -addext adds the wildcard SAN. OpenSSL Documentation

To inspect the resulting CSR:
~~~bash
#openssl req -in wildcard.eic.bnl.gov.csr -noout -text -verify
~~~

You should see:
~~~bash
Subject: CN = *.eic.bnl.gov
X509v3 Subject Alternative Name:
    DNS:*.eic.bnl.gov
~~~

----

**I. Host: `cdb.eic.bnl.gov` **

Service: Nginx 

| File | Central source | Installed path |
| --- | --- | --- |
| Wildcard certificate and chain | `/eic/data/ssl/wildcard.eic.bnl.gov.crt` | `/etc/nginx/ssl/cdb.eic.crt` |
| Matching private key | `/eic/data/ssl/wildcard.eic.bnl.gov.key` | `/etc/nginx/ssl/cdb.eic.key` |

## 1. Check the new files

Place the new certificate bundle and matching key at the central source paths above. Confirm the issuer, validity dates, and `*.eic.bnl.gov` SAN:

```bash
openssl x509 -in /eic/data/ssl/wildcard.eic.bnl.gov.crt \
  -noout -subject -issuer -dates -ext subjectAltName
```

The following two hashes must match. Stop if either command fails:

```bash
set -o pipefail
openssl x509 -in /eic/data/ssl/wildcard.eic.bnl.gov.crt -pubkey -noout |
  openssl pkey -pubin -outform DER | sha256sum

openssl pkey -in /eic/data/ssl/wildcard.eic.bnl.gov.key -pubout -outform DER |
  sha256sum
```

Inspect the bundle. It should contain the server certificate first, followed by its issuing intermediate(s):

```bash
openssl crl2pkcs7 -nocrl -certfile /eic/data/ssl/wildcard.eic.bnl.gov.crt |
  openssl pkcs7 -print_certs -noout
```

For the November 2026 replacement, the issuer was `SSL.com TLS Issuing RSA CA R1`, chaining to `SSL.com TLS RSA Root CA 2022`. The supplied bundle also included the root.

## 2. Back up the current files

Keep this shell open so `backup_stamp` remains available for rollback. Stop if either backup fails.

```bash
backup_stamp=$(date +%Y%m%d-%H%M%S)
cp -p /etc/nginx/ssl/cdb.eic.crt /etc/nginx/ssl/cdb.eic.crt.bak-"$backup_stamp"
cp -p /etc/nginx/ssl/cdb.eic.key /etc/nginx/ssl/cdb.eic.key.bak-"$backup_stamp"
```

## 3. Install and test

Run each command successfully before continuing:

```bash
install -o root -g root -m 644 /eic/data/ssl/wildcard.eic.bnl.gov.crt /etc/nginx/ssl/cdb.eic.crt
install -o root -g root -m 400 /eic/data/ssl/wildcard.eic.bnl.gov.key /etc/nginx/ssl/cdb.eic.key
restorecon -Rv /etc/nginx/ssl
nginx -t
```

Only if `nginx -t` succeeds:

```bash
systemctl reload nginx
systemctl is-active nginx
```

## 4. Verify the live site

```bash
openssl s_client -connect cdb.eic.bnl.gov:443 \
  -servername cdb.eic.bnl.gov -verify_hostname cdb.eic.bnl.gov \
  -verify_return_error </dev/null
```

Confirm the expected new CA chain and `Verify return code: 0 (ok)`. Check that the served leaf certificate matches the new file:

```bash
openssl x509 -in /eic/data/ssl/wildcard.eic.bnl.gov.crt -noout -fingerprint -sha256
openssl s_client -connect cdb.eic.bnl.gov:443 \
  -servername cdb.eic.bnl.gov </dev/null |
  openssl x509 -noout -fingerprint -sha256 -issuer -dates
```

The SHA-256 fingerprints must match. Open `https://cdb.eic.bnl.gov` and confirm the application works without certificate warnings.

## Rollback if needed

Use the backup timestamp from step 2; if using a new shell, set `backup_stamp` to that exact timestamp first. Restore both files, then test before reloading:

```bash
cp -p /etc/nginx/ssl/cdb.eic.crt.bak-"$backup_stamp" /etc/nginx/ssl/cdb.eic.crt
cp -p /etc/nginx/ssl/cdb.eic.key.bak-"$backup_stamp" /etc/nginx/ssl/cdb.eic.key
restorecon -Rv /etc/nginx/ssl
nginx -t && systemctl reload nginx
```

Repeat live verification after rollback. The old certificate may still have the trust issue that prompted replacement.
