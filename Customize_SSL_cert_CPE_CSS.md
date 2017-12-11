# Import customized certification into keystore files for SSL communication between the Content Platform Engine container and the Content Search Services container


Content Platform Engine and the Content Search Services images are built up with sample SSL configuration and sample self-signed certification. When customers need to run containers with existing customized certification, this customized certification need to be imported into the related keystore files for both the Content Platform Engine container and the Content Search Services container.

Please follow the certification and key setting:

- Public and private key algorithm is `RSA` and key size is `2048`
- Certification signature algorithm is `SHA256withRSA`
- Run keytool command with `IBM Java 1.8` version or above.


## 1. Import customized certification on the Content Search Services container

### Option A - Import customzed certification into sample CSS keystore file

A.1 In case the customized certification file name is cusCert.cer, and sample CSS keystore file is cssSelfsignedServerStore and the keystore password is "changeit". Mount path of cssSelfsignedServerStore is /opt/IBM/ContentSearchServices/CSS_Server/data. The alias value should not be same as existing ones in the keystore.

- ```keytool -importcert -alias cusalias -file cusCert.cer -keystore cssSelfsignedServerStore -storepass changeit```

A.2 Restart the Content Search Services containers
 

### Option B - Import customized certification into keystore file with different name and/or keystore password

B.1 Create the keystore file to contain the customized certification. If there are existing keystore file, ignore this step.

Command: ```keytool -genkey -v -keyalg RSA -keypass YourKeyPassword -keystore YourKeyStore -storepass YourStorePassword -validity NumberOfDays -dname "CN=YourHostName, OU=Unknown, O=Unknown, L=Unknown, ST=Unknown, C=Unknown" -keysize 2048```

e.g. ```keytool -genkey -v -keyalg RSA -keypass cusPassWord -keystore  cssCusStore -storepass cusPassWord -validity 3650 -dname "CN=CSS, OU=ECM, O=IBM, L=Unknown, ST=Unknown, C=Unknown" -keysize 2048```

B.2 In case the customized certification file name is cusCert.cer, and the target keystore file is cssCusStore and the keystore password is "cusPassWord". (Mount path of cssCusStore is /opt/IBM/ContentSearchServices/CSS_Server/data)

The keystore file is mounted to /opt/IBM/ContentSearchServices/CSS_Server/data/cssCusStore inside the Content Search Services container, and run the following command to import certification. (alias value should not be same as existing ones in the keystore)

- ```keytool -importcert -alias cusalias -file cusCert.cer -keystore cssCusStore -storepass cusPassWord```

B.3 Run the following command on the Docker host where the Content Search Services container is running 

```
docker exec -d [CSS container name] /usr/bin/css-dynamic-change.sh [CSS_KEY_STORE] [keyStoreName] [keyStorePassword]
e.g.
docker exec -d cssx1 /usr/bin/css-dynamic-change.sh CSS_KEY_STORE cssCusStore cusPassWord
```

## 2. Set customized certification on CPE container:
2.1 Create the keystores file to contain the customized certification. If there are existing keystore file, ignore this step.

- ```keytool -genkey -v -keyalg RSA -keypass cusPassWord -keystore  csskeystore.jks -storepass cusPassWord -validity 3650 -dname "CN=CPE, OU=ECM, O=IBM, L=Unknown, ST=Unknown, C=Unknown" -keysize 2048```
- ```keytool -genkey -v -keyalg RSA -keypass cusPassWord -keystore  csstruststore.jks -storepass cusPassWord -validity 3650 -dname "CN=CPE, OU=ECM, O=IBM, L=Unknown, ST=Unknown, C=Unknown" -keysize 2048```

2.2 Run the following command to import certification. (alias value should not be same as existing ones in the keystore)

- ```keytool -importcert -alias cusalias -file cusCert.cer -keystore csskeystore.jks -storepass cusPassWord```
- ```keytool -importcert -alias cusalias -file cusCert.cer -keystore csstruststore.jks -storepass cusPassWord```

2.3 Place these keystore files to one host directory with container folder mounted. For example:

```
/home/cpe_data/configDropins/overrides/csskeystore.jks
/home/cpe_data/configDropins/overrides/csstruststore.jks
```

2.4 The keystore password can be entered in clear text or encoded. The securityUtility encode option can be used to encode the password.

```
docker exec -it cpe /opt/ibm/wlp/bin/securityUtility encode cusPassWord
{xor}PCosDz4sLAgwLTs=
```

2.5 Create liberty SSL configuration XML file under liberty overrides host directory, please note that it requires using the full mount pathes inside container where the keystore files located in the SSL configuration XML file. For example:
/home/cpe_data/configDropins/overrides/ssl.xml

```
<?xml version="1.0" encoding="UTF-8"?>
<server>
    <!-- SSL keystore and truststore configuration for CSS -->
    <ssl id="cusSSLSettings"
        keyStoreRef="cusKeyStore"
        trustStoreRef="cusTrustStore"
        clientAuthenticationSupported="false"
        sslProtocol="TLSv1.2"
        securityLevel="CUSTOM"
        enabledCiphers="SSL_RSA_WITH_AES_128_CBC_SHA SSL_RSA_WITH_3DES_EDE_CBC_SHA SSL_RSA_FIPS_WITH_3DES_EDE_CBC_SHA SSL_RSA_WITH_AES_128_GCM_SHA2
56 SSL_RSA_WITH_AES_128_CBC_SHA256"
        />
    <keyStore id="cusKeyStore"
        location="/opt/ibm/wlp/usr/servers/defaultServer/configDropins/overrides/csskeystore.jks"
        type="JKS" password="{xor}PCosDz4sLAgwLTs=" />
    <keyStore id="cusTrustStore"
        location="/opt/ibm/wlp/usr/servers/defaultServer/configDropins/overrides/csstruststore.jks"
        type="JKS" password="{xor}PCosDz4sLAgwLTs=" />

</server>
```
