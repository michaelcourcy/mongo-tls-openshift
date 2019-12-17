# mongo-tls-openshift
This little project show how to adapt the official mongo image in openshift to run with tls mode. 

I found this tricky and should be shared. This is just an example deployment there is probably other way. Mongodb operator are anyway superseding this approach but it's still useful if you want to go without operator. 

## Let openshift create the certificate with the right CN and SAN 

We use the internal PKI of openshift : [openshift-service-serving-signer](https://docs.openshift.com/container-platform/3.11/dev_guide/secrets.html#service-serving-certificate-secrets).

In 01_mongodb-service we add the annotation `service.alpha.openshift.io/serving-cert-secret-name: nodejs-tls`

That creates a tls secret (mongodb-tls) with a CN and SAN related to the exposed service: 

```
oc get secret mongodb-tls -o "jsonpath={ .data['tls\.crt'] }" | base64 -d |openssl x509  -text 
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number: 4250756372739404458 (0x3afdb86647dc32aa)
    Signature Algorithm: sha256WithRSAEncryption
        Issuer: CN=openshift-service-serving-signer@1575279043
        Validity
            Not Before: Dec 16 13:26:48 2019 GMT
            Not After : Dec 15 13:26:49 2021 GMT
        Subject: CN=mongodb.my-project.svc
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (2048 bit)
                Modulus:
                    00:b0:fa:20:20:c9:9e:a2:44:d5:80:06:8d:28:6f:
                    92:20:ca:ad:25:b6:78:dd:f0:03:71:07:cf:16:e2:
                    16:1b:e3:bd:a0:5f:33:c8:5b:8e:2f:7d:96:f2:bb:
                    a4:4c:6e:ec:c0:5e:af:d9:5b:19:cc:7a:01:75:87:
                    ef:59:d8:77:16:e2:f0:69:3f:83:c7:84:4a:98:cb:
                    fe:6b:67:bc:8a:42:d3:52:e1:d7:30:e2:31:5c:7f:
                    0e:8f:11:82:e1:3a:c7:e8:8d:cc:0d:32:f9:ec:07:
                    83:5f:8d:66:e3:f1:41:3e:e0:64:6a:9e:3a:e4:0e:
                    72:cf:9d:f0:c2:61:32:a9:fc:d9:63:a8:4c:54:3d:
                    28:f4:13:8a:9c:38:b1:de:c5:08:2a:7a:86:a8:7b:
                    6e:2d:64:20:ba:f5:04:52:f4:5b:3e:e6:ee:64:ce:
                    95:c1:84:f1:80:ed:0f:a9:90:d7:7e:99:57:ce:8c:
                    aa:d9:81:6a:f8:c8:8c:00:d7:a4:56:c3:27:39:34:
                    35:c1:32:25:c8:9d:80:a3:ea:ca:f1:a0:e8:07:33:
                    6f:3e:6e:d3:e7:90:50:2a:a0:ac:5c:71:4c:2e:41:
                    4d:4c:cf:0b:f4:c6:53:b9:fc:4c:16:49:13:e6:6a:
                    10:47:e9:bb:5b:89:e8:88:ad:d0:7d:9e:e8:2a:8c:
                    ef:b1
                Exponent: 65537 (0x10001)
        X509v3 extensions:
            X509v3 Key Usage: critical
                Digital Signature, Key Encipherment
            X509v3 Extended Key Usage: 
                TLS Web Server Authentication
            X509v3 Basic Constraints: critical
                CA:FALSE
            X509v3 Subject Alternative Name: 
                DNS:mongodb.my-project.svc, DNS:mongodb.my-project.svc.cluster.local
            1.3.6.1.4.1.2312.17.100.2.1: 
                .$b6edc787-2007-11ea-8179-0a96d0769632
    Signature Algorithm: sha256WithRSAEncryption
         93:16:fb:58:41:22:58:29:c5:9b:13:61:37:7e:5d:79:a1:5b:
         a1:2f:c8:65:1a:a3:ba:f1:2d:44:07:ba:1f:93:d5:cb:d5:b0:
         6b:33:ac:51:44:b8:bf:71:07:de:c1:8d:45:fb:ce:ee:ab:87:
         7c:55:a5:5d:19:2b:37:94:27:7b:ca:be:cf:86:04:e5:b1:c6:
         39:93:d3:1a:1f:f7:f7:87:b2:2f:d4:3f:a2:08:c4:7b:e9:e2:
         d0:98:f0:14:f6:7f:67:9c:08:4c:0a:5f:d8:e7:e8:41:83:35:
         ea:2e:78:3e:91:fa:5e:9e:f6:a0:90:cb:30:a4:14:2e:a0:b2:
         e5:bc:a8:f7:bd:ba:0c:69:46:f0:0c:38:1c:f9:2e:ec:8b:5b:
         78:67:78:6b:3c:75:cb:00:9a:da:9a:ca:47:ec:0c:f8:06:fe:
         57:b5:c2:91:4b:1e:be:fb:48:fc:1f:f8:52:fc:6a:29:dd:14:
         ac:bf:59:b9:2b:6d:b0:5b:b8:87:cc:38:c0:49:1d:73:1a:27:
         46:3d:85:d0:a8:20:39:c2:59:ec:4d:8f:23:3e:c0:3e:bc:78:
         ed:e3:1f:01:3e:29:0a:22:ab:d5:74:d6:00:2f:83:42:9d:dd:
         61:48:1c:6f:a6:25:6c:93:4b:3b:2d:1c:b5:3f:92:46:9f:36:
         09:a2:b0:91
```

We see that we have a certificate signed for 2 years : 

```
            Not Before: Dec 16 13:26:48 2019 GMT
            Not After : Dec 15 13:26:49 2021 GMT
```

The CN and the SAN is properly defined : 

```
        Subject: CN=nodejs.nodejs-tls.svc

        ....

             X509v3 Subject Alternative Name: 
                DNS:mongodb.my-project.svc, DNS:mongodb.my-project.svc.cluster.local       
```

## Adding the CA in other pods

Other pods can trust cluster-created certificates (which are only signed for internal DNS names), by using the CA bundle in the /var/run/secrets/kubernetes.io/serviceaccount/service-ca.crt file that is automatically mounted in their pod.

## Use a pem key instead of a crt 

Mongod does not support defining the key and the crt in 2 different files thus once you use the serving sign service to generate the key 

```
    # Generate the tls for mongo 
    service.alpha.openshift.io/serving-cert-secret-name: mongodb-tls
    description: Exposes the database server
```

you need an init containers to build the pem

```
      initContainers:
      - name: create-mongodb-pem
        image: busybox:1.28
        command: ['sh', '-c', '-e', 'cat /etc/mongodb/tls/tls.key /etc/mongodb/tls/tls.crt > /etc/mongodb/pem/mongodb.pem']
```

## Redefine mongod.conf 

Impossible to owerwrite mongod.conf with a configmap without redefining 30-set-config-file.sh also. That's because 30-set-config-file.sh is writing in this file and a file mounted through a configmap is readonly. Thus we redefine 30-set-config-file.sh as well commenting the line that rewrite mongod.conf.

```
# Substitute environment variables in configuration file
# TEMP=`mktemp`; cp ${MONGODB_CONFIG_PATH} $TEMP; envsubst > ${MONGODB_CONFIG_PATH} < $TEMP
```

In mongod.conf we add the option that make mongod work with TLS. Notice allowSSL instead of requireSSL (which is proposed in the official doc) because in this case mongod is checking on himself without using tls, because it does not work (requireSSL) mongod decide to shutdown the server. That's a weird behaviour and I'm not completly sure that's coming from mongod or the author of the docker image.

```
        net:
          # Specify port number (27017 by default)
          port: 27017
          ssl:
            mode: allowSSL
            PEMKeyFile: /etc/mongodb/pem/mongodb.pem
            CAFile: /var/run/secrets/kubernetes.io/serviceaccount/service-ca.crt
            allowConnectionsWithoutCertificates: true
```

## Redefine readinessProbe to check ssl is working as well

That's not necessary to change the readinessProbe because we also allow non ssl connection but to be more complete we can check also the ssl connection work in the readiness probe. Not that in this case you need the --sslAllowInvalidHostnames to check on localhost because the vertificate does not have any SAN or CN with localhost.

```
        readinessProbe:
          exec:
            command:
            - /bin/sh
            - -i
            - -c
            - mongo --ssl --sslCAFile /var/run/secrets/kubernetes.io/serviceaccount/service-ca.crt 
                --sslAllowInvalidHostnames --host=localhost:27017 
                -u $MONGODB_USER -p $MONGODB_PASSWORD $MONGODB_DATABASE 
                --eval="quit()"
```
