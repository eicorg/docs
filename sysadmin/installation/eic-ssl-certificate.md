# EIC Servers Using SSL Certificate List

- ansible
- proxmox (prox01/02/03/04/05)
- kasm
- cdb
- hdb
- demo04

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

# Example Host 1: `cdb.eic.bnl.gov`

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


----

# Example Host 2: `ansible.eic.bnl.gov`
  
Doc Reference:  
https://github.com/eicorg/ansible-sys/blob/main/docs/z-appendix-api-ansible.md   
Step 8 - Configure Ansible AWX - Certificate

## 1. Confirm the TLS Secret

```bash
cd /home/eicuser/awx-on-k3s

kubectl get ingress awx-ingress -n awx \
  -o jsonpath='{.spec.tls}{"\n"}'
```

Confirm the ingress references `awx-secret-tls` for `ansible.eic.bnl.gov`.

## 2. Validate the Wildcard Certificate

Check validity and hostname coverage:

```bash
openssl x509 -in /eic/data/ssl/wildcard.eic.bnl.gov.crt \
  -noout -dates -checkhost ansible.eic.bnl.gov
```

Confirm the certificate and private key match:

```bash
openssl x509 -in /eic/data/ssl/wildcard.eic.bnl.gov.crt \
  -pubkey -noout | openssl sha256

openssl pkey -in /eic/data/ssl/wildcard.eic.bnl.gov.key \
  -pubout | openssl sha256
```

The two SHA256 values must be identical.

> The certificate file must contain the server certificate first,
> followed by the intermediate certificates in chain order.

## 3. Back Up the Existing Configuration

The secret backup contains the private key; keep it accessible only to root.

```bash
umask 077
TLS_BACKUP_DIR=$(mktemp -d /root/awx-tls-backup.XXXXXX)

kubectl get secret awx-secret-tls -n awx -o yaml \
  > "$TLS_BACKUP_DIR/awx-secret-tls.yaml"

cp -p base/tls.crt base/tls.key "$TLS_BACKUP_DIR/"

echo "Backup location: $TLS_BACKUP_DIR"
```

## 4. Update the Local Certificate Files

Keep the deployment files consistent with the live certificate:

```bash
install -m 644 /eic/data/ssl/wildcard.eic.bnl.gov.crt base/tls.crt
install -m 600 /eic/data/ssl/wildcard.eic.bnl.gov.key base/tls.key
```

Do not commit the private key to Git.

## 5. Update the Kubernetes TLS Secret

Update the existing secret without deleting it:

```bash
kubectl create secret tls awx-secret-tls -n awx \
  --cert=base/tls.crt \
  --key=base/tls.key \
  --dry-run=client -o yaml | kubectl apply -f -
```

Expected result:

```text
secret/awx-secret-tls configured
```

If Kubernetes reports a missing `last-applied-configuration` annotation,
it will add the annotation automatically.

Traefik loads the updated certificate automatically; no AWX or k3s restart
is normally required.

## 6. Verify HTTPS

Check the certificate served by the web interface:

```bash
openssl s_client \
  -connect ansible.eic.bnl.gov:443 \
  -servername ansible.eic.bnl.gov </dev/null 2>/dev/null |
  openssl x509 -noout -subject -issuer -dates -ext subjectAltName
```

Confirm:

- The certificate covers `ansible.eic.bnl.gov` through `*.eic.bnl.gov`.
- The issuer and validity dates match the installed wildcard certificate.
- [https://ansible.eic.bnl.gov](https://ansible.eic.bnl.gov) opens without a certificate warning.
- AWX login works.

> **Renewal recorded on November 19, 2026:**  
> Issuer: `SSL.com TLS Issuing RSA CA R1`  
> Expiration: `May 29, 2027`  
> Repeat this procedure when ITD provides the next wildcard certificate.

---

# Example Host 3: `prox01.eic.bnl.gov`

Doc Reference:  
https://github.com/eicorg/proxmox_production/blob/main/phase1-setup-installation.md   
11. Install Trusted TLS Certificate   

Follow existing procedure on prox01–prox05, copying the new certificate and matching key to /root/ on each node.   

For replacement, use:

~~~bash
pvenode cert set \
  /root/wildcard.eic.bnl.gov.crt \
  /root/wildcard.eic.bnl.gov.key \
  --force 1 --restart 1
~~~

Then verify on each node:

~~~bash
openssl s_client \
  -connect "$(hostname -f):8006" \
  -servername "$(hostname -f)" </dev/null 2>/dev/null |
  openssl x509 -noout -subject -issuer -dates
~~~

----
