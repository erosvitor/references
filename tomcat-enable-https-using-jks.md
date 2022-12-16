# Enabling HTTPS with JKS on Tomcat

## Introduction
The Hypertext Transfer Protocol Secure (HTTPS) is an extension of the Hypertext Transfer Protocol (HTTP). It is used for secure communication over a computer network, and is widely used on the Internet. In HTTPS, the communication protocol is encrypted using Transport Layer Security (TLS) or, formerly, Secure Sockets Layer (SSL).

Transport Layer Security (TLS) and its predecessor, Secure Sockets Layer (SSL), are technologies which allow web browsers and web servers to communicate over a secured connection. This means that the data being sent is encrypted by one side, transmitted, then decrypted by the other side before processing. This is a two-way process, meaning that both the server AND the browser encrypt all tra c before sending out data.

It is important to note that con guring Tomcat to take advantage of secure sockets is usually only necessary when running it as a stand-alone web server. When running Tomcat primarily as a Servlet/JSP container behind another web server, such as Apache or Microsoft IIS, it is usually necessary to con gure the primary web server to handle the SSL connections from users.

**IMPORTANT!** Java provides a relatively simple command-line tool, called "keytool", which can easily create a "self-signed" Certificate. Self-signed Certificates are simply user generated Certificates which have not been signed by a well-known CA (Certificate Authority) and are, therefore, not really guaranteed to be authentic at all. While self-signed certificates can be useful for some testing scenarios, they are not suitable for any form of production use.

## Java Keystore
A Java keystore is a repository where private keys and own identity certificate. This is typically a le and usually we'll use a keystore when we are a server and want to use HTTPS.

Until Java 8 the keystore format was JKS (Java KeyStore) and since Java 9 the default keystore format is PKCS12. The biggest difference between JKS and PKCS12 is that JKS is a format speci c to Java, while PKCS12 is a standardized and language-neutral way of storing encrypted private keys and certificates.

## Java Truststore
A Java truststore is used for the storage of certificates from the trusted Certificate Authority (CA), which is used in the veri cation of the certificate provided by the server in an SSL connection.

Java has bundled a truststore called 'cacerts' and it can be found in:

**Java 8**
```
JAVA_HOME/jre/lib/security
```

**Java 11**
```
JAVA_HOME/jre/lib/security
```

**Java 17**
```
JAVA_HOME/lib/security
```

To see all trusted Certificate Authorities stored in 'cacerts' run the following command:

```
$ JAVA_HOME/bin/keytool -list -keystore cacerts
```

We can override the default truststore location via the javax.net.ssl.trustStore property. Similarly, we can set javax.net.ssl.trustStorePassword and javax.net.ssl.trustStoreType to specify the truststore's password and type.

## Keystore versus Truststore
The work of Truststore is to verify the credentials, whereas the work of Keystore is to provide the credentials. These are the most important differences between Truststore and Keystore but not the only ones. These differences vary in Java and they are as follows:

- Truststore is used by Trust Manager and Keystore is used by Key Manager; they both perform different functions.
- Keystore contains private keys and is required only when a server is running on an SSL connection, whereas Truststore store public keys and the certificates issued form the certificate authority.
- To specify the path of a Keystore or Truststore, we need different extensions in Java. (Djavax.net.ssl.keyStore for Keystore and - Djavax.net.ssl.trustStore for Truststore.)
- There is also a difference in the password of the two (i.e. for Keystore it is given by the following extension Djavax.net.ssl.keyStorePassword and for truststore is given by Djavax.net.ssl.trustStorePassword).
- Keystore contains one private key for the host, while truststore contains zero private keys.

## Generate self-signed certificate
Tomcat currently operates only on JKS, PKCS11 or PKCS12 format keystores. The JKS format is Java's standard "Java KeyStore" format, and is the format created by the 'keytool' command-line utility. The PKCS12 format is an internet standard, and can be manipulated via OpenSSL.

**IMPORTANT!** Note that OpenSSL often adds readable comments before the key, but keytool does not support that. So if your certificate has comments before the key data, remove them before importing the certificate with keytool.

To create a self-signed certificate execute the following from a terminal command line:

```
$ JAVA_HOME/bin/keytool \
  -genkeypair \
  -alias <alias-name> \
  -keyalg RSA \
  -keypass <password> \
  -keystore <path>/<keystore-name>.jks \
  -storepass <password> \
  -dname "cn=<organization-name>,
          ou=<department-name>,
          o=<organization-name>,
          L=<city-name>,
          ST=<state-name>,
          c=<initials_country>"
```

Example
```
$ JAVA_HOME/bin/keytool \
  -genkeypair \
  -alias tomcatjks \
  -keyalg RSA \
  -keypass jks123 \
  -keystore ${user.home}/tomcatjks.jks \
  -storepass jks123 \
  -dname "cn=XYZ Enterprise,
          ou=Engineering,
          o=XYZ Enterprise,
          L=Curitiba,
          ST=Parana,
          c=BR"
```

**Tip:** The RSA algorithm should be preferred as a secure algorithm, and this also ensures general compatibility with other servers and components.

To list details about JKS keystore execute the following from a terminal command line:

```
$ JAVA_HOME/bin/keytool -list -v -keystore <path>/<keystore-name>.jks
```

Example
```
$ JAVA_HOME/bin/keytool -list -v -keystore ~/tomcatjks.jks
```

## Tomcat 10 Settings
**Step 1**
Locate the following section in the file <tomcat-path>/conf/server.xml
```
<!-- Define an SSL/TLS HTTP/1.1 Connector on port 8443 with HTTP/2
     This connector uses the NIO implementation. The default
     SSLImplementation will depend on the presence of the APR/native
     library and the useOpenSSL attribute of the AprLifecycleListener.
     Either JSSE or OpenSSL style configuration may be used regardless of
     the SSLImplementation selected. JSSE style configuration is used below.
-->
<!--
<Connector port="8443" protocol="org.apache.coyote.http11.Http11NioProtocol"
           maxThreads="150" SSLEnabled="true">
    <UpgradeProtocol className="org.apache.coyote.http2.Http2Protocol" />
    <SSLHostConfig>
        <Certificate certificateKeystoreFile="conf/localhost-rsa.jks"
                     type="RSA" />
    </SSLHostConfig>
</Connector>
-->
```

**Step 2**
Remove the following comments
```
<!--
<Connector port="8443" protocol="org.apache.coyote.http11.Http11NioProtocol"
           maxThreads="150" SSLEnabled="true">
    <UpgradeProtocol className="org.apache.coyote.http2.Http2Protocol" />
    <SSLHostConfig>
        <Certificate certificateKeystoreFile="conf/localhost-rsa.jks"
                     type="RSA" />
    </SSLHostConfig>
</Connector>
-->
```

**Step 3**
Change the connector as below
```
<Connector port="8443" protocol="org.apache.coyote.http11.Http11NioProtocol"
           maxThreads="150" SSLEnabled="true">
    <UpgradeProtocol className="org.apache.coyote.http2.Http2Protocol" />
    <SSLHostConfig>
        <Certificate certificateKeystoreFile="${user.home}/tomcatjks.jks"
                     certificateKeystorePassword="jks123"
                     type="RSA" />
    </SSLHostConfig>
</Connector>
```

**Step 4**
Verify the access
```
https://localhost:8443
```

## Redirect HTTP request to HTTPS
Add the following lines at end of <tomcat-path>/conf/web.xml file
```
<security-constraint>
  <web-resource-collection>
    <web-resource-name>Entire Application</web-resource-name>
    <url-pattern>/*</url-pattern>
  </web-resource-collection>
  <user-data-constraint>
    <transport-guarantee>CONFIDENTIAL</transport-guarantee>
  </user-data-constraint>
</security-constraint>
```

## References
https://tomcat.apache.org/tomcat-10.0-doc/ssl-howto.html
https://tomcat.apache.org/tomcat-9.0-doc/ssl-howto.html
https://tomcat.apache.org/tomcat-8.0-doc/ssl-howto.html
https://tomcat.apache.org/tomcat-7.0-doc/ssl-howto.html
