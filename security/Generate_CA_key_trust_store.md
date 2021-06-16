# SSL

### Steps to generate CA, Truststore and Keystore

**Please Note** - For Truststores and Keystores, I have shown the steps only to generate *
kafka.zookeeper.truststore.jks* and *kafka.zookeeper.keystore.jks*. The procedure is the same for generating such files
for brokers, producers, and consumers.

**1. Generate CA** <br />
openssl req -new -x509 -keyout ca-key -out ca-cert -days 3650

**2. Create Truststore** <br />
keytool -keystore kafka.zookeeper.truststore.jks -alias ca-cert -import -file ca-cert

**3. Create Keystore** <br />
keytool -keystore kafka.zookeeper.keystore.jks -alias zookeeper -validity 3650 -genkey -keyalg RSA -ext SAN=dns:
localhost

````
Warning:
The JKS keystore uses a proprietary format. It is recommended to migrate to PKCS12 which is an industry standard format using "keytool -importkeystore -srckeystore kafka.zookeeper.keystore.jks -destkeystore kafka.zookeeper.keystore.jks -deststoretype pkcs12".
````

**4. Create certificate signing request (CSR)** <br />
keytool -keystore kafka.zookeeper.keystore.jks -alias zookeeper -certreq -file ca-request-zookeeper

**5. Sign the CSR** <br />
openssl x509 -req -CA ca-cert -CAkey ca-key -in ca-request-zookeeper -out ca-signed-zookeeper -days 3650 -CAcreateserial

**6. Import the CA into Keystore** <br />
keytool -keystore kafka.zookeeper.keystore.jks -alias ca-cert -import -file ca-cert

**7. Import the signed certificate from step 5 into Keystore** <br />
keytool -keystore kafka.zookeeper.keystore.jks -alias zookeeper -import -file ca-signed-zookeeper

### To check all the brokers connected to the Zookeeper running in 2-way SSL mode

2-way SSL mode

zookeeper client certificates creation

```
zookeeper_shell.sh
```

keytool -keystore zookeeper.client.truststore.jks -alias ca-cert -import -file ca-cert keytool -keystore
zookeeper.client.keystore.jks -alias zookeeper-client -validity 3650 -genkey -keyalg RSA -ext SAN=dns:localhost keytool
-keystore zookeeper.client.keystore.jks -alias zookeeper-client -certreq -file ca-request-zookeeper-client openssl x509
-req -CA ca-cert -CAkey ca-key -in ca-request-zookeeper-client -out ca-signed-zookeeper-client -days 3650
-CAcreateserial keytool -keystore zookeeper.client.keystore.jks -alias ca-cert -import -file ca-cert keytool -keystore
zookeeper.client.keystore.jks -alias zookeeper-client -import -file ca-signed-zookeeper-client

```
server.properties
```

keytool -keystore kafka.broker0.truststore.jks -alias ca-cert -import -file ca-cert keytool -keystore
kafka.broker0.keystore.jks -alias broker0 -validity 3650 -genkey -keyalg RSA -ext SAN=dns:localhost keytool -keystore
kafka.broker0.keystore.jks -alias broker0 -certreq -file ca-request-broker0 openssl x509 -req -CA ca-cert -CAkey ca-key
-in ca-request-broker0 -out ca-signed-broker0 -days 3650 -CAcreateserial keytool -keystore kafka.broker0.keystore.jks
-alias ca-cert -import -file ca-cert keytool -keystore kafka.broker0.keystore.jks -alias broker0 -import -file
ca-signed-broker0


