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
