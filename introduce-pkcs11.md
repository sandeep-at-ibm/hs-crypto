---

copyright:
  years: 2019, 2021
lastupdated: "2021-08-11"

keywords: hsm, cloud hsm, tke cli, pkcs11, PKCS11 library, cryptographic operations, cryptographic functions, PKCS 11

subcollection: hs-crypto

---


{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}
{:important: .important}
{:note: .note}
{:external: target="_blank" .external}
{:term: .term}

# Introducing PKCS #11
{: #pkcs11-intro}

PKCS #11 is a standard that specifies an application programming interface (API), called *Cryptoki*, for devices that hold cryptographic information and perform cryptographic functions. The Cryptoki API follows a simple object-based approach. The approach addresses the goals of technology independence and resource sharing, presenting to applications a common, logical view of the device called a *cryptographic token*.

Cryptoki isolates an application from the details of the cryptographic device. The application does not have to change the interface to a different type of device or to run in a different environment. Thus, the application is portable. The functions of the Cryptoki API are organized into the following categories:

* [General-purpose functions](http://docs.oasis-open.org/pkcs11/pkcs11-base/v2.40/os/pkcs11-base-v2.40-os.html#_Toc416959740){: external} (four functions)
* [Slot and token management functions](http://docs.oasis-open.org/pkcs11/pkcs11-base/v2.40/os/pkcs11-base-v2.40-os.html#_Toc416959741){: external} (nine functions)
* [Session management functions](http://docs.oasis-open.org/pkcs11/pkcs11-base/v2.40/os/pkcs11-base-v2.40-os.html#_Toc416959742){: external} (eight functions)
* [Object management functions](http://docs.oasis-open.org/pkcs11/pkcs11-base/v2.40/os/pkcs11-base-v2.40-os.html#_Toc416959743){: external} (nine functions)
* [Encryption functions](http://docs.oasis-open.org/pkcs11/pkcs11-base/v2.40/os/pkcs11-base-v2.40-os.html#_Toc416959744){: external} (four functions)
* [Decryption functions](http://docs.oasis-open.org/pkcs11/pkcs11-base/v2.40/os/pkcs11-base-v2.40-os.html#_Toc416959745){: external} (four functions)
* [Message digesting functions](http://docs.oasis-open.org/pkcs11/pkcs11-base/v2.40/os/pkcs11-base-v2.40-os.html#_Toc416959746){: external} (five functions)
* [Signing, verifying, and MAC functions](http://docs.oasis-open.org/pkcs11/pkcs11-base/v2.40/os/pkcs11-base-v2.40-os.html#_Toc416959747){: external} (12 functions)
* [Dual-purpose cryptographic functions](http://docs.oasis-open.org/pkcs11/pkcs11-base/v2.40/os/pkcs11-base-v2.40-os.html#_Toc416959748){: external} (four functions)
* [Key management functions](http://docs.oasis-open.org/pkcs11/pkcs11-base/v2.40/os/pkcs11-base-v2.40-os.html#_Toc416959749){: external} (five functions)
* [Random number generation functions](http://docs.oasis-open.org/pkcs11/pkcs11-base/v2.40/os/pkcs11-base-v2.40-os.html#_Toc416959750){: external} (two functions)
* [Parallel function management functions](http://docs.oasis-open.org/pkcs11/pkcs11-base/v2.40/os/pkcs11-base-v2.40-os.html#_Toc416959751){: external} (two functions)

Not all PKCS #11 functions are implemented by {{site.data.keyword.cloud}} {{site.data.keyword.hscrypto}}. For the implemented PKCS #11 functions, see [Supported PKCS #11 functions](/docs/hs-crypto?topic=hs-crypto-pkcs11-api-ref#pkcs11_function_list).
{: note}

To review the PKCS #11 standard documentation, see:
* [Cryptographic Token Interface Usage Guide Version 2.40](http://docs.oasis-open.org/pkcs11/pkcs11-ug/v2.40/pkcs11-ug-v2.40.html){: external}
* [Cryptographic Token Interface Base Specification Version 2.40 Plus Errata 01](http://docs.oasis-open.org/pkcs11/pkcs11-base/v2.40/pkcs11-base-v2.40.html){: external}
* [Cryptographic Token Interface Current Mechanisms Specification Version 2.40 Plus Errata 01](http://docs.oasis-open.org/pkcs11/pkcs11-curr/v2.40/pkcs11-curr-v2.40.html){: external}

## PKCS #11 implementation components
{: #pkcs11-components}

To connect and use the PKCS #11 API, you need to understand the PKCS #11 API that is implemented by {{site.data.keyword.hscrypto}} and the relationship with the GREP11 API. For more information, see [Comparing the PKCS #11 API with the GREP11 API](/docs/hs-crypto?topic=hs-crypto-introduce-cloud-hsm#compare-grep11-pkcs11). With the support of the PKCS #11 API, you don't need to change your existing applications that use the PKCS #11 standard. {{site.data.keyword.hscrypto}} also provides the isolated keystores to store cryptographic keys generated by the PKCS #11 functions. These keys are protected by the master key and the applications never see the key files locally.

Before you can use the PKCS #11 API, first install the PKCS #11 library. In this way, the PKCS #11 application can interact with the PKCS #11 library, which then calls cryptographic functions that are implemented by {{site.data.keyword.hscrypto}} through gRPC. The following diagram shows the key components that are implemented by the {{site.data.keyword.hscrypto}} PKCS #11 library and the interactions among different components.

![Performing cryptographic operations with the PKCS #11 API](/images/pkcs-components.svg "Performing cryptographic operations with the PKCS #11 API"){: caption="Figure 1. Performing cryptographic operations with the PKCS #11 API" caption-side="bottom"}

The following sections explain each PKCS #11 component in detail.

### Application
{: #pkcs11-application-intro}

An application runs within a single address space. All threads of control are running in the application. An application becomes a Cryptoki application by calling the Cryptoki function `C_Initialize` from one of the threads. After this call is made, the application can call other Cryptoki functions. When the application finishes using Cryptoki, it calls the Cryptoki function `C_Finalize` and ceases to be a Cryptoki application.

### User
{: #pkcs11-user-intro}

The PKCS #11 standard defines two types of users for login: a security officer (SO) and a normal user. If a user does not log in using the `C_Login` Cryptoki function, the user is also known as an anonymous user. Only the normal user can access private objects on the token, and that access is granted only after the normal user is authenticated. In the {{site.data.keyword.hscrypto}} PKCS #11 implementation, security officers and normal users are authenticated by API keys. Anonymous users are also supported. For instructions on setting up PKCS #11 user types, see [Best practices for setting up PKCS #11 user types](/docs/hs-crypto?topic=hs-crypto-best-practice-pkcs11-access).

### Session
{: #pkcs11-session-intro}

The Cryptoki API requires that an application opens one or more sessions with a token to gain access to the objects and functions of the token. A session provides a logical connection between the application and the token. A session can be a read/write (R/W) session or a read-only (R/O) session.

To gain access to the private objects of the token, the normal user needs to log in and be authenticated. See [PKCS #11 Cryptographic Token Interface Usage Guide](http://docs.oasis-open.org/pkcs11/pkcs11-ug/v2.40/cn02/pkcs11-ug-v2.40-cn02.html#_Toc406759989){: exteral} for an in-depth look at sessions.

### Key object
{: #pkcs11-object-intro}

Key objects are stored in either the in-memory keystore that resides with the PKCS #11 application or in a database-backed keystore. If the CKA_TOKEN attribute is set to `true` for the key object, the key object is stored in the database-backed keystore. Otherwise, the key object is stored in the in-memory keystore. 

As shown in the following diagram, a PKCS #11 key object is an example of a PKCS #11 object class:

* **Data**: A data object is defined by an application. 
* **Certificates**: A certificate object stores a certificate. 
* **Keys**: A key object stores a cryptographic key. The key can be a public key, a private key, or a secret key. Each type of these keys has subtypes for use in specific mechanisms.
    * Public key: The public component of a key pair that is used by anyone to encrypt messages intended for a particular recipient that has access to the private key of the key pair. The public key is also used to verify signatures created by the private key.
    * Private key: The private component of a key pair that is used to decrypt messages. The private key is also used to create signatures.
    * Secret key: A secret key is a generated stream of bits that is used to encrypt and decrypt messages symmetrically.

![PKCS #11 object classes](/images/object-class.svg "PKCS #11 object classes"){: caption="Figure 2. PKCS #11 object classes" caption-side="bottom"}

The data and certificate object classes are not supported in this implementation.
{: note}

### {{site.data.keyword.iamshort}}
{: #pkcs11-iam-intro}

IBM {{site.data.keyword.iamshort}} (IAM) provides user authentication and access control for the implementation of the PKCS #11 API. By using API keys, the PKCS #11 library obtains bearer tokens that are used to perform Cryptoki API calls. 

This implementation of PKCS #11 equates an API key with a user's PIN. For more information about setting up the service ID and the corresponding API key, see [Create service IDs for the SO user, normal user, and anonymous user](/docs/hs-crypto?topic=hs-crypto-best-practice-pkcs11-access#step2-create-service-id-api-key).
{: note}

### Crypto service
{: #pkcs11-crypto-intro}

As part of the PKCS #11 library initialization process, a gRPC connection is made from the PKCS #11 library to the {{site.data.keyword.cloud_notm}}. The gRPC connection facilitates the PKCS #11 library to call Cryptoki functions, such as `C_Encrypt`, `C_Decrypt`, `C_Sign`, and `C_Verify`, which requires the use of a Hardware Security Module (HSM).

### Keystore
{: #pkcs11-keystore-intro}

Two major types of keystores are available:

* **In-memory keystores**: Stores key objects temporarily in memory. Key objects that are stored in the in-memory keystore are also known as *session objects*. Session objects in a specific session are destroyed when you call the `C_CloseSession` function for that session. Session objects in all sessions are destroyed after the `C_Finalize` function is called.
* **Database-backed keystores**: Stores key objects in databases. Key objects that are stored in the database-backed keystore are also known as *token objects*. If the `sessionauth` parameter is enabled and a password for the keystore is configured, the database-backed keystore is encrypted and authenticated. For each service instance, a maximum of five authenticated keystores are supported. You can enable the `sessionauth`  parameter to encrypt the generated keys into the keystore or to decrypt the key before you use it. The password can be 6 - 8 characters.

Keystore passwords are not stored in the service instance. You, as the keystore administrator, are responsible for maintaining a local copy of the passwords. If a password is lost, you need to contact the Support team to reset the keystore, which means all data in the keystore is cleared.
{: note}

Both the in-memory keystores and the database-backed keystores are composed of the following types of keystores:

* **Public keystore**: Stores keys that are less sensitive and that are to be accessed by any user types.
* **Private keystore**: Stores keys that encrypt sensitive data and that are to be accessed by normal users only.

Depending on the [user types](#pkcs11-user-intro) and key attributes settings, the generated keys are stored in different keystores. The following list summarizes the criteria of how keys are stored:

* The CKA_TOKEN attribute value decides whether the generated key is stored in an in-memory keystore or in a database-backed keystore.

    If you want to store the key in the database-backed keystore, set CKA_TOKEN to `TRUE` in key or key pair generation templates. The PKCS #11 library initialization process establishes a gRPC connection with the {{site.data.keyword.cloud_notm}}, which facilitates the storing and retrieval of key objects to or from the database-backed keystore. By default, CKA_TOKEN is set to `FALSE`, which means the key object is stored in an in-memory keystore within the process address space of the PKCS #11 application.

* The CKA_PRIVATE attribute value decides whether a key that is generated by normal users is to be stored in a public or private keystore.

    By default, if a user is logged in as a normal user, generated keys are stored in the private keystores, except for the case where CKA_PRIVATE is set to `FALSE`. If a user is logged in as an SO user or is not logged in (known as an anonymous user), then generated keys are always stored in the public keystores. If an SO user or an anonymous user specifies CKA_PRIVATE to `TRUE` in the key generation templates, an error is returned from the server.

    For an asymmetric key pair, you need to set the CKA_PRIVATE attribute separately for both the public and private keys, which means the key pairs can be stored in different keystores.

Refer to the following table for detailed explanations of the relationship between user types, key attributes, and keystores, and how keys are stored.

| User type | CKA_TOKEN | CKA_PRIVATE | Keystore for keys |Private or public?|
| --------- | ----------------------- | -----------------------| ----------------------- |---------------------------- |
| SO user| `FALSE`  |N/A  |   In-memory keystore  |Public |
| SO user| `TRUE`  | N/A  |Database-backed keystore  |Public |
| Anonymous user | `FALSE`  |N/A   | In-memory keystore  |Public |
| Anonymous user | `TRUE`  | N/A   | Database-backed keystore  | Public|
| Normal user   | `FALSE` |`FALSE`  | In-memory keystore |Public |
| Normal user   | `FALSE` |`TRUE`  |In-memory keystore | Private |
| Normal user   |`TRUE` |`FALSE`  |  Database-backed keystore  |Public|
| Normal user   | `TRUE` |`TRUE`  | Database-backed keystore  |Private |
{: caption="Table 1. Cases of storing EP11 keys in different keystores" caption-side="bottom"}
