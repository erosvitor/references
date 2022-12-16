# Enabling HTTPS with PCKS12 on Tomcat

## Generating a self-signed certificate PKCS12

**Step 1:** Generate Private Key and Certificate Signing Requests (CSR)

```
$ openssl req -newkey rsa:2048 -nodes -keyout <private-key-name>.key -x509 -days 365 -out <certificate-name>.csr

example:

$ openssl req -newkey rsa:2048 -nodes -keyout pk100.key -x509 -days 365 -out certificate100.csr
```

**Step 2:** Review the CSR

```
$ openssl x509 -text -noout -in <certificate-name>.csr

example:

$ openssl x509 -text -noout -in certificate100.csr
```

**Step 3:** Combine your Private Key and CSR in a PKCS12 (P12) bundle

```
$ openssl pkcs12 -inkey <private-key-name>.key -in <certificate-name>.csr -export -out <certificate-name>.p12

example:

$ openssl pkcs12 -inkey pk100.key -in certificate100.csr -export -out certificate100.p12
```

**Step 4:** Validate your P2 file

```
$ openssl pkcs12 -in <certificate-name>.p12 -noout -info

example:

$ openssl pkcs12 -in certificate100.p12 -noout -info
```

## Tomcat settings

**Step 1:** Edit the JAVA_HOME/lib/security/java.security file and change the following line
```
keystore.type=jks

  to

keystore.type=pkcs12
```

**Tip:** This change is necessary only JDK 8 or early

**Step 2:** Configure the SSL connector by editing the Tomcat server.xml
```
<Connector port="8443"
  maxThreads="150"
  minSpareThreads="25"
  maxSpareThreads="75"
  enableLookups="false"
  disableUploadTimeout="true"
  acceptCount="100"
  debug="0"
  scheme="https"
  secure="true"
  clientAuth="false"
  sslProtocol="TLS" 
  keystoreType="PKCS12"
  keystoreFile="<full-path>/certificate.p12"
  keystorePass="<password>"
  truststoreType="PKCS12"
  truststoreFile="<full-path>/certificate.p12"
  truststorePass="<password>"
 />
```
