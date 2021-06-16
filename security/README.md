# Kafka security

using PLAINTEXT in communication allows anyone can read data in between.

In config/server.properties

```
listeners=PLAINTEXT://localhost:9092
```

**Communication can be encrypted using SSL/TLS protocol.**

<<IMAGES>>

## SSL based Encrytion

SSL/TLS is a transport layer protocol which encrypts the data in the producer side and decrypts the data in brokers

* Communication between clients and brokers
* inter broker communation
* broker <--> zookeeper communication

SSL based encryption needs Certificates

Work flow:

1.Create CA - certificate Authority 2.Broker will have KeyStore which contains private key and certificate. 3.broker
sends a certificate signing request to CA 4.CA signs the certificate. 5.clients(Producer and consumer) will have
TrustStore with them .trustStore will contais root CA details. 6.when clients tries to connect to kafka brokers .brokers
send the signed certificate to client. 7.client verifies whether the signed CA is in TrustStore.is not it thorws
Exception. 8.if it is a valid certificate .Using the cerificate clients will encryt the messages and SSL comminucation
will happen between client and broker.

in consumer side if the config is set

```
security.protocol=SSL
```

and if we try to connect to normal port in broker side below Error will come

```
Connection to node -1 (localhost/127.0.0.1:9092) terminated during authentication. This may happen due to any of the following reasons: (1) Authentication failed due to invalid credentials with brokers older than 1.0.0, (2) Firewall blocking Kafka TLS traffic (eg it may only allow HTTPS traffic), (3) Transient network issue.
```

so broker side also we need to listen using SSL instead of PLAINTEXT.

## Authentication

Authentication can be done by both SSL(2 way SSL) AND SASL

in 2 way authentication both the parties must authenticate themself so both of them need to have key store and trust
store.

trust store contains the CA certificate key store contains singed-private key by CA and CA certificate.

first we will see zookeeper and zookeeper client using 2 way authentication using SSL

muralimanoj@muralimanoj-Laptop:~/Desktop/kafka/bin$ ./zookeeper-shell.sh localhost:2182 -zk-tls-config-file
/home/muralimanoj/Desktop/kafka/config/use_case_2/cluster1/zookeeper-client.properties

if u want SSL to authenticate use

```
security.protocol=SSL
```

if u want SASL

```
security.protocol=SASL_SSL
```

SASL has many mechanism to authenticate

* plain (this is different from PLAINTEXT)
* kerberos protocol
* SCRAM 256/512

First we will see SASL_PLAIN mechanism with SSL

In Clients side

1. SASL for Authentication and SSL for Encryption

security.protocol=SASL_SSL

2. As we are using SASL to authticate instead of 2-way SSL client does not need Key store proprities

ssl.truststore.location=/var/ssl/private/kafka.client.truststore.jks ssl.truststore.password=test1234

3. Set the SASL mechanism (we are using PLAIN )and JAAS config

sasl.mechanism=PLAIN sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required \
username="client" \
password="client-secret";

So the full config:

# security protocol SASL_SSL means SASL for authentication SSL for Encryption

security.protocol=SASL_SSL

# What TLS version going to use

ssl.protocol=TLSv1.3

# type of trust store file whether JKS or PKCS12

ssl.truststore.type = JKS

# only trust store file need as authentication done by SASL

# if it is SSL 2-way authentication means need to provide keystore config also

ssl.truststore.location=/var/ssl/private/kafka.client.truststore.jks ssl.truststore.password=test1234

# SASL authentication mechanism PLAIN OR SCRAM-SHA-512 OR GSSAPI (Kerberos)

sasl.mechanism=PLAIN

# JAAS(java authentication and authorization service - simple APIs can determine the identity) config  with PLAIN username and password

sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required \
username="murali" \
password="manoj1234";

To communicate via SSL/TLS we need to below to KeyStore and a TrustStore

These files are password protected and stored in the file system where application is running These files are stored in
two formats 1.jks (Default format until java 8- java specific)
2.pkcs12 (Default format after java 9 - language neutral)

KetStore A Java keystore stores private key entries, certificates with public keys or just secret keys. Usually, we'll
use a keystore when we are a server and want to use HTTPS. During an SSL handshake, the server looks up the private key
from the keystore and presents its corresponding public key and certificate to the client.

In 2-way authentication if the client also needs to authenticate itself – a situation called mutual authentication –
then the client also has a keystore and also presents its public key and certificate.

TrustStore

If a client talks to a Java-based server over HTTPS, the server will look up the associated key from its keystore and
present the public key and certificate to the client. We, the client, then look up the associated certificate in our
truststore. If the certificate or Certificate Authorities presented by the external server is not in our truststore,
we'll get an SSLHandshakeException and the connection won't be set up successfully.

client -------------------------------------->  Server
[ TrustStore - list of CA ]                                             [KeyStore - private key and certificate]

                <---------------------------------------
                    [Public key (certificate) ]

Links:

https://docs.confluent.io/platform/current/security/security_tutorial.html#generating-keys-certs

https://www.youtube.com/watch?v=U0XennY3_Ac&t=2954s

https://www.youtube.com/watch?v=hR_OuiqLgOo

## Authorization

ACL command to add permission for a User

1.We can add or alter or remove ACLS 2.we can provide allow-principal or deny-principal for particular User (
User:<username> User is case sensitive)
3.mention the operation allowed or denied 4.mention the topic

```
./kafka-acls.sh \
--zk-tls-config-file ../config/cluster1/zookeeper_cluster1.properties  \
--authorizer-properties zookeeper.connect=localhost:2181 \
--add \ 
--allow-principal User:murali-producer \
--operation WRITE --operation DESCRIBE \
--topic ssl-topic

./kafka-acls.sh \
--zk-tls-config-file ../config/cluster1/zookeeper_cluster1.properties  \
--authorizer-properties zookeeper.connect=localhost:2181 \
--add \
--allow-principal User:murali-consumer \
--operation READ --operation DESCRIBE \
--topic ssl-topic
```




