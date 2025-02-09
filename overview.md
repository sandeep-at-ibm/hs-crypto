---

copyright:
  years: 2018, 2021
lastupdated: "2021-11-08"

keywords: key management, dedicated key management, hsm, hardware security module, cloud hsm, dedicated hsm, keep your own key, kyok, cryptographic operation, key storage, encryption key, cloud encryption, encryption at rest, secure service container, ssc

subcollection: hs-crypto

---


{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}
{:important: .important}
{:external: target="_blank" .external}
{:term: .term}
{:video: .video}

# Overview
{: #overview}

{{site.data.keyword.cloud}} {{site.data.keyword.hscrypto}} is a dedicated key management service and [Hardware Security Module (HSM)](#x6704988){: term} that provides you with the Keep Your Own Key capability for cloud data encryption. Built on FIPS 140-2 Level 4 certified hardware, {{site.data.keyword.hscrypto}} provides you with exclusive control of your encryption keys.
{: shortdesc}

Watch the following video to learn how {{site.data.keyword.hscrypto}} provides you with exclusive encryption key control and data protection in the cloud:

![IBM Cloud Hyper Protect Crypto Services Explainer](https://www.kaltura.com/p/1773841/sp/177384100/embedIframeJs/uiconf_id/27941801/partner_id/1773841?iframeembed=true&entry_id=1_zxwh6ixm){: video output="iframe" data-script="none" id="mediacenterplayer" frameborder="0" width="560" height="315" allowfullscreen webkitallowfullscreen mozAllowFullScreen}

## Why {{site.data.keyword.cloud_notm}} {{site.data.keyword.hscrypto}}?
{: #why_hpcs}

Data and information security is crucial and essential for IT environments. As more data moves to the cloud, keeping data protected becomes a non-trivial challenge. Built on IBM LinuxONE technology, {{site.data.keyword.hscrypto}} helps ensure that only you have access to your keys and data. A single-tenant key management service with key vaulting that is provided by dedicated customer-controlled HSMs helps you easily create and manage your encryption keys. Alternatively, you can bring your own encryption keys to the cloud. The service uses the same key-provider API as {{site.data.keyword.keymanagementserviceshort}}, a multi-tenant key management service, to provide a consistent approach to adopting {{site.data.keyword.cloud_notm}} services.

{{site.data.keyword.hscrypto}} is a dedicated HSM that is controlled by you. {{site.data.keyword.cloud_notm}} administrators have no access. The service is built on FIPS 140-2 Level 4-certified hardware, the highest offered by any cloud provider in the industry. IBM is the first to provide cloud command-line interface (CLI) for HSM [master key](#x2908413){: term} initialization to help enable you to take ownership of the cloud HSM. You can also load the master key with the {{site.data.keyword.IBM_notm}} {{site.data.keyword.hscrypto}} Management Utilities. The Management Utilities create and store your master key parts on smart cards and never exposes your secrets to the workstation and cloud, thus ensuring the highest level of protection to your secrets.

{{site.data.keyword.hscrypto}} can integrate with {{site.data.keyword.cloud_notm}} data and storage services as well as VMware&reg; vSphere&reg; and VSAN, for providing data-at-rest encryption.

The managed cloud HSM supports the industry-standard cryptographic operations by using the Public-Key Cryptography Standards (PKCS) #11. You don't need to change your existing applications that use PKCS #11 standard to make it run in the {{site.data.keyword.hscrypto}} environment. The PKCS #11 library accepts the PKCS #11 API requests from your applications and remotely accesses the cloud HSM to execute the corresponding cryptographic functions, such as digital signing and validation.

Enterprise PKCS #11 over gRPC (GREP11) is also supported by {{site.data.keyword.hscrypto}}. The EP11 library provides an interface similar to the industry-standard [PKCS #11 application programming interface (API)](http://docs.oasis-open.org/pkcs11/pkcs11-base/v2.40/os/pkcs11-base-v2.40-os.html){: external}.

With the built-in encryption of {{site.data.keyword.hscrypto}}, you can easily build cloud applications with sensitive data. {{site.data.keyword.hscrypto}} provides you with complete control of your data and encryption keys, including the master key. The service also helps your business meet regulatory compliance with the technology that provides exclusive controls on the external and privileged user access to data and keys.





## How does {{site.data.keyword.hscrypto}} work?
{: #how-hpcs-work}

The following items are a few highlights of the {{site.data.keyword.hscrypto}} architecture:
- Applications connect to {{site.data.keyword.hscrypto}} through the PKCS #11 API or the GREP11 API.
- Dedicated keystore in {{site.data.keyword.hscrypto}} is provided to ensure data isolation and security. Privileged users are locked out for protection against abusive use of system administrator credentials or root user credentials.
- Secure Service Container (SSC) provides the enterprise level of security and impregnability that enterprise customers expect from [IBM LinuxONE](https://www.ibm.com/it-infrastructure/linuxone){: external} technology.
- FIPS 140-2 Level 4 compliant cloud HSM is enabled for highest physical protection of secrets.

For an architectural diagram of {{site.data.keyword.hscrypto}}, see [Service architecture, workload isolation, and dependencies](/docs/hs-crypto?topic=hs-crypto-architecture-workload-isolation).

## Key features
{: #key-features}

{{site.data.keyword.hscrypto}} provides both key management and cloud HSM functions:

### Key management service
{: #key-management}

* **Key lifecycle management**

    {{site.data.keyword.hscrypto}} provides a single-tenant key management service to create, import, rotate, and manage keys with the standardized API. After the encryption keys are deleted, you can be assured that your data is no longer retrievable.

* **Encryption for {{site.data.keyword.cloud_notm}} data and workload services**

    By integrating with other {{site.data.keyword.cloud_notm}} services, {{site.data.keyword.hscrypto}} offers the capability of bringing your own encryption to the cloud. The service provides double-layer protection for your cloud data by wrapping the encryption keys that are associated with your cloud services.

* **Access management and auditing**

    {{site.data.keyword.hscrypto}} integrates with {{site.data.keyword.iamshort}} (IAM) to enable your granular control over user access to service resources. For more information, see [Managing user access](/docs/hs-crypto?topic=hs-crypto-manage-access).

    You can also monitor and audit events and activities of {{site.data.keyword.hscrypto}} by using {{site.data.keyword.at_full_notm}}. For more information, see [Auditing events for {{site.data.keyword.hscrypto}}](/docs/hs-crypto?topic=hs-crypto-at-events).

### Cloud hardware security module
{: #cloud-hsm}

* **Customer-controlled HSM**

    With Keep Your Own Key, you can take the ownership of the HSM through assigning your own administrators and loading master keys with {{site.data.keyword.hscrypto}}. This ensures your full control of the entire key hierarchy with no access even from {{site.data.keyword.cloud_notm}} administrators.

* **Cryptographic operations**

    {{site.data.keyword.hscrypto}} supports the standard PKCS #11 API and the Enterprise PKCS #11 over gRPC (GREP11) API for cryptographic operations. The operations include generating keys, encrypting and decrypting data, signing data, and verifying signatures. The cryptographic functions are executed in HSMs and can be accessed through APIs to provide hardware-based protection for your applications.

* **Security certification**

    The service is built on FIPS 140-2 Level 4-certified hardware, the highest security level that is offered in the industry. The HSM is also certified to meet the Common Criteria Part 3 conformant EAL 4.



## What's next
{: #overview-next}

- To get an overall tutorial about using {{site.data.keyword.hscrypto}}, check out [Getting started with {{site.data.keyword.cloud_notm}} {{site.data.keyword.hscrypto}}](/docs/hs-crypto?topic=hs-crypto-get-started).
- To find out more about programmatically managing your keys, check out the [{{site.data.keyword.hscrypto}} key management API reference doc](/apidocs/hs-crypto){: external}.
- To find out more about the PKCS #11 API, see [Introducing PKCS #11](/docs/hs-crypto?topic=hs-crypto-pkcs11-intro) and [PKCS #11 API reference](/docs/hs-crypto?topic=hs-crypto-pkcs11-api-ref).
- To find out more about the GREP11 API, see [Introducing EP11 over gRPC](/docs/hs-crypto?topic=hs-crypto-grep11_intro) and [GREP11 API reference](/docs/hs-crypto?topic=hs-crypto-grep11-api-ref).
- For more information about the compliance certificates that {{site.data.keyword.hscrypto}} receives, see [Security and compliance](/docs/hs-crypto?topic=hs-crypto-security-and-compliance).
- To find more about available {{site.data.keyword.cloud_notm}} services for integration, see [Integrating {{site.data.keyword.cloud_notm}} services with {{site.data.keyword.hscrypto}}](/docs/hs-crypto?topic=hs-crypto-integrate-services).
