X.509
=====

From Ristic, Bulletproof SSL and TLS, p.382

Blog post on X.509 certificate (from CA or self-signed)
About 
1) how to create a chain of a root-ca and a sub-ca
2) create CA certificates
3) create a self-signed certificate
4) Use a free CA authority: Let's Encrypt 

Using a certificate is a good way to prevent “man-in-the-middle” attacks. (see Schneider about man-in-the-midde attack)

## Root Certification Authority (CA) Generation
 
1) Generate key and Certificate Signing Request (CSR) 

$ openssl req -new -config root-ca.conf -out root-ca.csr -keyout private/root-ca.k

PEM pass phrase: roma caput mundi

2) Create self-signed certificate
 
$ openssl ca -selfsign -config root-ca.conf -in root-ca.csr -out root-ca.crt -extensions ca_ext

Enter pass phrase: roma caput mundi

3) Generate a Certificate Revocation List (CRL)

$ openssl ca -gencrl -config root-ca.conf -out root-ca.crl

Enter pass phrase for ./private/root-ca.key: roma caput mundi

## Root CA Operations

1) Issue a certificate for a sub certification authority

$ openssl ca -config root-ca.conf -in sub-ca.csr -out sub-ca.crt -extensions sub_ca_ext

2) Revoke a certificate of a sub certification authority

$ openssl ca -config root-ca.conf -revoke certs/1002.pem -crl_reason keyCompromise

## Create a certficate for Online Certificate Signing Protocol (OCSP) signing

1) Create key and CSR 

$ openssl req -new -newkey rsa:2048 -subj "/C=GB/O=Example/CN=OCSP Root Responder" -keyout private/root-ocsp.key -out root-ocsp.csr

Enter PEM pass phrase: roma caput mundi

2) Issue a certificate

$ openssl ca -config root-ca.conf -in root-ocsp.csr -out root-ocsp.crt -extensions ocsp_ext -days 30

3) Start the OCSP responder

$ openssl ocsp -port 9080 -index db/index -rsigner root-ocsp.crt -rkey private/root-ocsp.key -CA root-ca.crt -text

4) Test the OCSP server from another shell

$ openssl ocsp -issuer root-ca.crt -CAfile root-ca.crt -cert root-ocsp.crt -url http://127.0.0.1:9080

## Creating a Subordinate CA

1) Prepare a configuration file for the subordinate CA using the root CA configuration file as a template

2) Subordinate CA generation: generate key and CSR

$ openssl req -new -config sub-ca.conf -out sub-ca.csr -keyout private/sub-ca.key

3) Issue a certificate for the subordinate CA by the root CA

$ openssl ca -config root-ca.conf -in sub-ca.csr -out sub-ca.crt -extensions sub_ca_ext

## Subordinate CA Operations

1) Issue a server certificate (missing server.csr) 

a) Create a private key for the server

$ openssl genpkey -algorithm RSA -out private/server.key -pkeyopt rsa_keygen_bits:2048

b) Generate the Certificate Signing Request (CSR) for the server to be sent to the subauthority

$ openssl req -new -key private/server.key -out server.csr

c) The subauthority signs a certificate for the server from the request

$ openssl ca -config sub-ca.conf -in server.csr -out server.crt -extensions server_ext

2) Issue a client certificate (missing client.csr)

a) Create a private key for the client

$ $ openssl genpkey -algorithm RSA -out private/client.key -pkeyopt rsa_keygen_bits:2048

b) Generate the Certificate Signing Request (CSR) for the client to be sent to the subauthority 

$ openssl req -new -key private/client.key -out client.csr

c) The subauthority signs a client certificate from the request
 
$ openssl ca -config sub-ca.conf -in client.csr -out client.crt -extensions client_ext


## Signing Your Own Certificate

You might want to create a self-signed certificate without sending any request to a certification authority. For example a certificate 
for the server

$ openssl req -new -x509 -days 365 -key private/server.key -out server.crt

## Examining your certificate

$ openssl x509 -text -in server.crt -noout

A self-signed certificate will have the property
 X509v3 Basic Constraints:
                CA:TRUE

meaning that it is a CA certificate and could be used to sign new certificates. Usually certificates issued by a CA have this property set 
to FALSE as the subscriber will not be able to sign other certificates using the one issued by the CA.
In a self-signed certificate the property Subject Key Identifier and Authority Key Identifier have the same values while in a CA-signed 
certificate the Authority Key Identifier value must be the same as the Subject Key Identifier value of the issuing CA certificate.
