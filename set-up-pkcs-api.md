---

copyright:
  years: 2020, 2021
lastupdated: "2021-08-12"

keywords: set up api, pkcs api, pkcs11 library, cryptographic operations, use pkcs11 api, access pkcs api, pkcs11, cryptographic functions

subcollection: hs-crypto

---

{:shortdesc: .shortdesc}
{:codeblock: .codeblock}
{:screen: .screen}
{:pre: .pre}
{:tip: .tip}
{:note: .note}
{:important: .important}
{:external: target="_blank" .external}

# Performing cryptographic operations with the PKCS #11 API
{: #set-up-pkcs-api}

{{site.data.keyword.cloud}} {{site.data.keyword.hscrypto}} provides the standard PKCS #11 API to access the {{site.data.keyword.hscrypto}} cloud HSM for cryptographic operations.
{: shortdesc}

## Prerequisites
{: #prerequisite-pkcs-api}

Before you can set up and use the PKCS #11 API, first follow the [Best practices for setting up PKCS #11 user types](/docs/hs-crypto?topic=hs-crypto-best-practice-pkcs11-access) to create different service ID API keys for the various PKCS #11 user types.

## Step 1: Set up the PKCS #11 library
{: #step1-setup-pkcs-library}

You need to set up the PKCS #11 library on your workstation to make it available for your applications to call the standard PKCS #11 functions.

The PKCS #11 library, for both the amd64 and s390x platforms, is supported only on Linux&reg;.
{: note}

1. [Download the latest PKCS #11 library](https://github.com/IBM-Cloud/hpcs-pkcs11/releases){: external}. The library file names use the naming convention: pkcs11-grep11-<**platform**>.so.<**version**>. The platform is either *amd64* or *s390x* and the version is the standard *major.minor.build* syntax.
2. Move the library into a folder that is accessible by your applications. For example, if you are running your application on Linux, you can move the library to `/usr/local/lib`, `/usr/local/lib64`, or `/usr/lib`.

## Step 2: (Optional) Verify the integrity and authenticity of the PKCS #11 library
{: #step2-verify-pkcs-library}

For maximum security, verify the integrity and authenticity of the PKCS #11 library before you run your PKCS #11 applications to use the library.

{{site.data.keyword.hscrypto}} enables [signed code verification](https://en.wikipedia.org/wiki/Code_signing){: external} to ensure that the signature matches the original code. If the downloaded PKCS #11 library file is altered or corrupted, a different signature is produced and the verification fails. To make sure that the files are not tampered with or corrupted during the download process, complete the following steps by using the [OpenSSL command-line tool](https://wiki.openssl.org/index.php/Binaries){: external}.

1. Download the latest version of the following files from the [library repository](https://github.com/IBM-Cloud/hpcs-pkcs11/releases){: external} to the same directory where you store the PKCS #11 library:

    - `pkcs11-grep11-<platform>.so.<version>.sig`: The signed cryptographic hash of the PKCS #11 library, where platform is either *amd64* or *s390x* and version is the *major.minor.build* of the signature file. Both **platform** and **version** must match the respective **platform** and **version** of the PKCS #11 library that you use.

    - `signing_cert.pem`: The signing certificate of the {{site.data.keyword.hscrypto}} PKCS #11 client files.

    - `digicert_cert.pem`: An intermediate code signing certificate to prove the signing certificate of the {{site.data.keyword.hscrypto}} PKCS #11 client files.

2. Extract the public key from the signing certificate `signing_cert.pem` to the `sigkey.pub` file with the following command:

    ```
    openssl x509 -pubkey -noout -in signing_cert.pem -out sigkey.pub
    ```
    {: pre}

3. Verify the integrity of the PKCS #11 library file with the following command:

    ```
    openssl dgst -sha256 -verify sigkey.pub -signature pkcs11-grep11-<platform>.so.<version>.sig pkcs11-grep11-<platform>.so.<version>
    ```
    {: pre}

    Replace **platform** with either *amd64* or *s390x* and replace **version** with the *major.minor.build* of the library.

    When the verification is successful, `Verified OK` is displayed.

4. Verify the authenticity and validity of the signing certificate with the following command:

    ```
    openssl ocsp -no_nonce -issuer digicert_cert.pem -cert signing_cert.pem -VAfile digicert_cert.pem -text -url http://ocsp.digicert.com -respout ocsptest
    ```
    {: pre}

    When the verification is successful, `Response verify OK` and `signing_cert.pem: good` are displayed in the output.

5. If the verification fails, cancel the installation and [contact IBM for support](/docs/hs-crypto?topic=hs-crypto-getting-help).

## Step 3: Set up the PKCS #11 configuration file
{: #step3-setup-configuration-file}

In order to connect the PKCS #11 library to the {{site.data.keyword.hscrypto}} cloud HSM to perform cryptographic functions, you need to complete the following steps to set up the configuration file.

1. Create a configuration file that is named `grep11client.yaml` based on the following example. The [library repository](https://github.com/IBM-Cloud/hpcs-pkcs11/releases){: external} also provides a template for you to adapt. You can refer to the comments in the code to understand each field.

    ```yaml
    iamcredentialtemplate: &defaultiamcredential
              enabled: true
              endpoint: "https://iam.cloud.ibm.com"
              # Keep the 'apikey' empty. It will be overridden by the Anonymous user API key configured later.
              apikey:
              # The Universally Unique IDentifier (UUID) of your Hyper Protect Crypto Services instance.
              instance: "<instance_id>"

    tokens:
      0:
        grep11connection:
          # The EP11 endpoint address starting from 'ep11'. For example: "ep11.us-south.hs-crypto.cloud.ibm.com"
          address: "<EP11_endpoint_URL>"
          port: "<EP11_endpoint_port_number>" # The EP11 endpoint port number
          tls:
            enabled: true # EP11 requires TLS connection.
            # Set it 'true' if you want to enable mutual TLS connections.
            # By default, set it 'false' because EP11 requires server-only authentication.
            mutual: <enable_mtls>
            # 'cacert' is a full-path certificate file. In Linux with the 'ca-ca-certificates' package installed, this is normally not needed.
            cacert:
            # Specify the file path of the client certificate if you enable mutual TLS. Otherwise, keep it empty.
            certfile: <client_certificate>
            # Specify the file path of the client certificate private key if you enable mutual TLS. Otherwise, keep it empty.
            keyfile: <client_certificate_private_key>
        storage:
            # 'remotestore' needs to be enabled if you want to generate keys with the attribute CKA_TOKEN.
          remotestore:
            enabled: true
        users:
          0: # The index of the Security Officer (SO) user MUST be 0.
            # The name for the Security Officer (SO) user. For example: "Administrator".
            name: "<SO_user_name>"
            # NEVER put the API key under the SO user for security reasons.
            iamauth:
              <<: *defaultiamcredential
          1: # The index of the normal user MUST be 1.
            # The name for the normal user. For example: "Normal user".
            name: "<normal_user_name>"
            # The 128-bit UUID of the private keystore. For example: "f00db2f1-4421-4032-a505-465bedfa845b".
            tokenspaceID: "<private_keystore_spaceid>"
            # NEVER put the API key under the normal user for security reasons.
            iamauth:
              <<: *defaultiamcredential
            sessionauth:
              enabled: true # Enable this option to encrypt and authenticate the keystore.
              # Authenticated keystore password; must be 6-8 characters in length
              tokenspaceIDPassword: "<private_keystore_password>"
          2: # The index of the anonymous user MUST be 2.
            # The name for the anonymous user. For example: "Anonymous".
            name: "<anonymous_user_name>"
            # The 128-bit UUID of the public keystore. For example: "ca22be26-b798-4fdf-8c83-3e3a492dc215".
            tokenspaceID: "<public_keystore_spaceid>"
            iamauth:
              <<: *defaultiamcredential
              # The API key for the anonymous user. It will overide the 'apikey' in the previous defaultcredentials.iamauth.apikey field
              apikey: "<apikey_for_anonymous_user>"
            sessionauth:
              enabled: false
              tokenspaceIDPassword: # Authenticated keystore password; must be 6-8 characters in length
    logging:
      # Set the logging level.
      # The supported levels, in an increasing order of verboseness: 'panic', 'fatal', 'error', 'warning'/'warn', 'info', 'debug', 'trace'. The Default value is 'warning'.
      loglevel: "<logging_level>"
      logpath: "<log_file_path>" # The full path of your logging file.

    ```
    {: codeblock}

    Replace the variables in the example according to the following table:

    <table>
      <tr>
        <th>Variable</th>
        <th>Description</th>
      </tr>
      <tr>
        <td><em>instance_id</em></td>
        <td>The unique identifier that is assigned to your {{site.data.keyword.hscrypto}} service instance. You can get the identifier through <strong>Overview</strong> &gt; <strong>Instance</strong> in the {{site.data.keyword.cloud_notm}} console, or you can retrieve it with the {{site.data.keyword.cloud_notm}} Resource Controller API. For detailed instructions, see <a href="/docs/hs-crypto?topic=hs-crypto-retrieve-instance-ID">Retrieving an instance ID</a>.</td>
      </tr>
      <tr>
        <td><em>EP11_endpoint_URL</em></td>
        <td>The {{site.data.keyword.hscrypto}} Enterprise PKCS #11 (EP11) API endpoint. You can get it through <strong>Overview</strong> &gt; <strong>Connect</strong> &gt; <strong>EP11 endpoint URL</strong> in the {{site.data.keyword.cloud_notm}} console, or you can dynamically <a href="/apidocs/hs-crypto#getinstance" target="_blank">retrieve the endpoint URL</a> with the API. Depending on whether you are using a public or <a href="/docs/hs-crypto?topic=hs-crypto-secure-connection">private network</a>, use the public or private EP11 endpoint URL. </td>
      </tr>
      <tr>
        <td><em>EP11_endpoint_port_number</em></td>
        <td>The port number of the EP11 API endpoint. It is located after the colon in the endpoint URL.</td>
      </tr>
      <tr>
        <td><em>enable_mtls</em></td>
        <td>Valid values are <code>true</code> or <code>false</code> to indicate whether you want to enable mutual TLS to add a second layer of authentication for PKCS #11 API access. By default, set it <code>false</code> as EP11 requires server-only authentication. For more information about the mutual TLS connections, see <a href="/docs/hs-crypto?topic=hs-crypto-enable-authentication-ep11">Enabling the second layer of authentication for EP11 connections</a>.</td>
      </tr>
      <tr>
        <td><em>client_certificate</em></td>
        <td>If you enable mutual TLS connections, specify the file path of the client certificate that is uploaded to your instance by the certificate administrator. Otherwise, keep this field empty.</td>
      </tr>
      <tr>
        <td><em>client_certificate_private_key</em></td>
        <td>If you enable mutual TLS connections, specify the file path of the client certificate private key that is used to sign the certificate. Otherwise, keep this field empty.</td>
      </tr>
      <tr>
        <td><em>SO_user_name</em></td>
        <td>The name for the Security Officer (SO) user type. The PKCS #11 standard defines two types of users for login: the security officer (SO) and the normal user. For more information about the PKCS #11 user types, see <a href="http://docs.oasis-open.org/pkcs11/pkcs11-ug/v2.40/cn02/pkcs11-ug-v2.40-cn02.html#_Toc406759984" target="_blank">PKCS #11 Cryptographic Token Interface Usage Guide Version 2.40 - Users</a>.</td>
      </tr>
      <tr>
        <td><em>normal_user_name</em></td>
        <td>The name for the normal user type. The PKCS #11 standard defines two types of users for login: the security officer (SO) and the normal user. For more information about the PKCS #11 user types, see <a href="http://docs.oasis-open.org/pkcs11/pkcs11-ug/v2.40/cn02/pkcs11-ug-v2.40-cn02.html#_Toc406759984" target="_blank">PKCS #11 Cryptographic Token Interface Usage Guide Version 2.40 - Users</a>.</td>
      </tr>
      <tr>
        <td><em>private_keystore_spaceid</em></td>
        <td>
        <p>The 128-bit <a href="https://www.cryptosys.net/pki/uuid-rfc4122.html" target="_blank">Universally Unique IDentifier (UUID)</a> of the private keystore. You can generate the UUID with a third-party tool, such as <a href="https://www.uuidgenerator.net/" target="_blank">UUID generator</a>.</p> <p>{{site.data.keyword.hscrypto}} provides you with two database-backed EP11 keystores for enhanced security and better user access management: the private keystore that only the normal user type can access and the public keystore that all user types can access.</p>
        </td>
      </tr>
      <tr>
        <td><em>private_keystore_password</em></td>
        <td>Authorized sessions can be used by enabling the <code>sessionauth</code> configuration option. If the <code>sessionauth</code> option is enabled, a text password that is 6-8 characters in length is required for the <code>tokenspaceIDPassword</code> field. Authorized sessions are specific to the HSM and are used in the PKCS #11 flow to login and logout, and they are required for authenticated key operations. All keys generated using authorized sessions are stored in an authenticated and encrypted keystore. The <code>tokenspaceIDPassword</code> field is used to protect the keys in an authenticated and encrypted keystore. For each service instance, a maximum of five authenticated keystores are supported.</td>
      </tr>
      <tr>
        <td><em>anonymous_user_name</em></td>
        <td>The name for the anonymous user. The PKCS #11 standard defines two types of users for login: the security officer (SO) and the normal user. If a user does not log in by using the <code>C_Login</code> Cryptoki function, then the user is known as an anonymous user. For more information about the PKCS #11 user types, see <a href="http://docs.oasis-open.org/pkcs11/pkcs11-ug/v2.40/cn02/pkcs11-ug-v2.40-cn02.html#_Toc406759984" target="_blank">PKCS #11 Cryptographic Token Interface Usage Guide Version 2.40 - Users</a>.</td>
      </tr>
      <tr>
        <td><em>public_keystore_spaceid</em></td>
        <td>
          <p>The 128-bit <a href="https://www.cryptosys.net/pki/uuid-rfc4122.html" target="_blank">Universally Unique IDentifier (UUID)</a> of the public keystore. You can generate the UUID with a third-party tool, such as <a href="https://www.uuidgenerator.net/" target="_blank">UUID generator</a>.</p> <p>{{site.data.keyword.hscrypto}} provides you with two database-backed EP11 keystores for enhanced security and better user access management: the private keystore that only the normal user type can access and the public keystore that all user types can access.</p>
        </td>
      </tr>
      <tr>
        <td><em>apikey_for_anonymous_user</em></td>
        <td>
          The service ID API key that you create for the anonymous user type in the [previous prerequisites step](#prerequisite-pkcs-api).
        </td>
      </tr>
      <tr>
        <td><em>logging_level</em></td>
        <td>The supported logging levels, in an increasing order of verboseness: <code>panic</code>, <code>fatal</code>, <code>error</code>, <code>warning</code>/<code>warn</code>, <code>info</code>, <code>debug</code>, and <code>trace</code>. The Default value is <code>warning</code>. For more information about the logging levels, see <a href="http://www.thejoyofcode.com/Logging_Levels_and_how_to_use_them.aspx" target="_blank">Logging levels</a>.</td>
      </tr>
      <tr>
        <td><em>log_file_path</em></td>
        <td>The full path of your logging file. It saves all the logs that are generated when your applications interact with the {{site.data.keyword.hscrypto}} cloud HSM to execute PKCS #11 functions.</td>
      </tr>
      <caption>Table 1. Describes the variables that are needed to create the PKCS #11 configuration file.</caption>
    </table>

    To encrypt and authenticate the keystore that is used by PKCS #11, enable the `sessionauth` parameter and configure the password for the keystore. For each service instance, a maximum of five authenticated keystores are supported. The password can be 6-8 characters. The keystore passwords are not stored in the service instance. You, as the keystore administrator, are responsible for maintaining a local copy of the passwords. If a password is lost, you need to contact IBM Support to reset the keystore, which means all data in the keystore is cleared.
    {: note}

2. Move the configuration file into the same directory as the application (for example, pkcs11-tool) that uses the PKCS #11 library. Optionally, the PKCS #11 configuration file can be placed in the `/etc/ep11client` directory. Create the `/etc/ep11client` directory if it does not exist.

## Step 4: Use the PKCS #11 library to make PKCS #11 API calls
{: #step4-use-pkcs-library}

After you set up the library and the configuration file, the keystores must be initialized. To initialize the keystores, the security officer (SO) user needs to perform a `C_InitToken` operation.
{: important}

After the keystores are initialized, use the PKCS #11 library to call the standard PKCS #11 functions to generate, store, and list keys. For the detailed list of supported PKCS #11 functions, see [PKCS #11 API reference](/docs/hs-crypto?topic=hs-crypto-pkcs11-api-ref).

Depending on features and security requirements of your application, pass different service ID API keys that you created in the [previous prerequisites step](#prerequisite-pkcs-api) so that your applications can perform the corresponding operations. For example, if your application needs to delete a keystore, provide the SO user API key. If your application needs to access the private keystore to store new keys, you need to provide the normal user API key. For more information about user access management for the PKCS #11 API, see [Best practices for setting up PKCS #11 user types](/docs/hs-crypto?topic=hs-crypto-best-practice-pkcs11-access).

If you are running a Java PKCS #11 application on the IBM Z (s390x) platform, it is suggested to use the OpenJ9 JVM. You can [download the JVM here](https://github.com/AdoptOpenJDK/openjdk11-binaries/releases/download/jdk-11.0.8%2B10_openj9-0.21.0/OpenJDK11U-jdk_s390x_linux_openj9_linuxXL_11.0.8_10_openj9-0.21.0.tar.gz){: external}.
{: note}

## What's next
{: #set-up-pkcs-api-next-steps}

- Check out the tutorial that shows how to [use {{site.data.keyword.hscrypto}} PKCS #11 library for Oracle Database Transparent Database Encryption](/docs/hs-crypto?topic=hs-crypto-tutorial-tde-pkcs11) to better understand the PKCS #11 library usage.
- Check out the [PKCS #11 API reference](/docs/hs-crypto?topic=hs-crypto-pkcs11-api-ref) for the detailed information about cryptographic functions.
