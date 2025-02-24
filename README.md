# If your LDAP certificate is in DER format, you’ll need to convert it before importing it into a keystore or truststore. Here’s what to do:

1. Check if the Certificate is in DER Format

Run this command:

openssl x509 -inform DER -in ldap_cert.der -text -noout

	•	If it shows readable details, it’s a valid DER certificate.
	•	If you see an error, the file might be invalid or in another format.

2. Convert DER to PEM

Tomcat and keytool typically require PEM format. Convert your .der file to .pem:

openssl x509 -inform DER -in ldap_cert.der -out ldap_cert.pem

3. Import the Certificate into the Java Truststore (If Needed)

If the certificate is for trusting the LDAP server, add it to Java’s cacerts truststore:

keytool -import -trustcacerts -keystore $JAVA_HOME/lib/security/cacerts -storepass changeit -noprompt -alias ldap_cert -file ldap_cert.pem

On Windows:

"C:\Program Files\Java\jdk-21\bin\keytool" -import -trustcacerts -keystore "C:\Program Files\Java\jdk-21\lib\security\cacerts" -storepass changeit -noprompt -alias ldap_cert -file C:\path\to\ldap_cert.pem

Then, verify:

keytool -list -keystore $JAVA_HOME/lib/security/cacerts -storepass changeit | grep ldap_cert

4. Convert DER to PKCS12 Keystore (If You Need a Keystore)

If you need a keystore with both a certificate and private key, but your .der file is just a certificate (no key), you must obtain the private key first.

If you do have the private key (ldap_key.der), convert both to PKCS12:

openssl pkcs12 -export -in ldap_cert.pem -inkey ldap_key.pem -out keystore.p12 -name tomcat -password pass:yourpassword

Then update application.properties:

server.ssl.key-store=classpath:keystore.p12
server.ssl.key-store-type=PKCS12
server.ssl.key-store-password=yourpassword
server.ssl.key-alias=tomcat

5. Restart and Test

After making these changes:
	•	Restart your Spring Boot application.
	•	If you see SSL handshake errors, enable debug logging:

-Djavax.net.debug=ssl


	•	Check for certificate issues in logs.

Summary

Scenario	Steps
LDAP Server Requires Trust Only	Convert .der to .pem, import into cacerts
LDAP Server Requires Client Authentication	Convert .der to .pem, ensure you have the private key, create a .p12 keystore, update Spring Boot configs

Let me know if you need further clarification!