---

copyright:
  years: 2019, 2021
lastupdated: "2021-08-16"

keywords: smart card, smart card reader, install driver, linux, trusted key entry, tke, master key, initialize service, load master key

subcollection: hs-crypto

---


{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}
{:tip: .tip}
{:note: .note}
{:important: .important}
{:external: target="_blank" .external}
{:term: .term}

# Setting up smart cards and the Management Utilities
{: #prepare-management-utilities}

With smart cards and the [{{site.data.keyword.IBM}} {{site.data.keyword.hscrypto}} Management Utilities](/docs/hs-crypto?topic=hs-crypto-initialize-instance-mode#understand-management-utilities), you can initialize service instances with the highest level of security. The Management Utilities use smart cards for storing [signature keys](/docs/hs-crypto?topic=hs-crypto-understand-concepts#signature-key-concept) and [master key parts](/docs/hs-crypto?topic=hs-crypto-understand-concepts#master-key-part-concept).
{: shortdesc}

The following diagram gives you an overview of steps you need to take to initialize service instances with smart cards and the Management Utilities. This topic covers the steps to set up the Management Utilities. For the detailed instructions on initialize the service instance, see [Initializing service instances by using smart cards and the Management Utilities](/docs/hs-crypto?topic=hs-crypto-initialize-hsm-management-utilities).

![The task flow of service instance initialization with smart cards and the Management Utilities](/images/hsm_initialization_flow_smartcard.svg "The task flow of service instance initialization with smart cards and the Management Utilities"){: caption="Figure 1. Task flow of service instance initialization with smart cards and the Management Utilities" caption-side="bottom"}

## Step 1: Order smart cards and smart card readers
{: #order-smart-card-and-reader}

Before you can configure smart cards, you need to first order [smart cards](/docs/hs-crypto?topic=hs-crypto-understand-concepts#smart-card-concept) and [smart card readers](/docs/hs-crypto?topic=hs-crypto-understand-concepts#smart-card-reader-concept) online.

### Ordering smart cards
{: #order-smart-card}

Complete the following steps to order smart cards:

1. On the [{{site.data.keyword.IBM_notm}} Maintenance Parts Retail shop web page](https://www-store.shop.ibm.com/shop/en-US/partsusretail){: external}, enter Field Replaceable Unit (FRU) part number **00RY790** in the search box.

    Online smart card ordering is only available in the United States. For procurement from other countries, see [FAQ: How can I procure smart cards and smart card readers?](/docs/hs-crypto?topic=hs-crypto-faq-provisioning-operations#faq-procure-smart-card).
    {: note}

2. Enter the quantity of packages. Each package contains a set of two smart cards.

    A minimum of two smart cards is needed to use the Management Utilities: a certificate authority smart card and an Enterprise PKCS #11 (EP11) smart card. The certificate authority smart card defines a set of smart cards that can work together. This set of smart cards is called a smart card zone. EP11 smart cards hold a signature key and one or more master key parts that are used to configure service instances. Currently, the Management Utilities support 2 or 3 master key parts to be loaded.

    For added security, the 2 or 3 master key parts that are used to configure a service instance can be generated on separate EP11 smart cards that are assigned to different people. The signature key used to sign commands to the crypto module can be generated on a separate EP11 smart card. Three or four EP11 smart cards would then be needed to configure a service instance.

    It is also suggested to [create backup copies of all used smart cards](/docs/hs-crypto?topic=hs-crypto-faq-provisioning-operations#faq-smart-card-setup) and to save the backup smart cards in a secure place. For maximum security, 10 smart cards would be needed, including backup smart cards (two certificate authority smart cards and eight EP11 smart cards).

3. Click **Add to Current Order** and continue to check out.

### Ordering smart card readers

The supported smart card reader type is SPR332 v2.0 Secure Class 2 PIN Pad Reader (part number 905127-1). The following are a few third-party online shops where you can order a smart card reader. The deliver policy might vary depending on your geographical locations:

* [Identiv](https://shop.identiv.com/products/spr332-v2-0-secure-class-2-pin-pad-reader){: external}
* [Digifeat](https://digifeat.com/product/spr332-v2-0/){: external}
* [SCM PC-Card GmbH](https://www.scm-pc-card.de/all-products/101/spr332-v2){: external}
* [Amazon](https://www.amazon.de/SPR332-Speed-Secure-Kartenleser-Online-Abrechnung/dp/B0092NGGWW){: external}

## Step 2: Install the smart card reader driver
{: #install-smart-card-reader-driver}

You need to install the Identiv SPR332 V2 smart card reader driver on your local workstation. Currently, only Red Hat Enterprise Linux&reg; 8.0.0 is supported.

You need to take the [smart card considerations](/docs/hs-crypto?topic=hs-crypto-define-smart-card-security-policy) into account when you plan your security policy for your workstation and smart card readers. Otherwise, your smart cards might be exposed to some vulnerabilities.
{: note}



Before you install the smart card reader driver on a Linux operating system, download and extract the driver package from the [Identiv SPR332 v2.0 Support website](https://support.identiv.com/spr332/){: external}. And then, install the smart card reader driver by completing the following steps:



- Red Hat Enterprise Linux&reg; 8.0.0

    1. Install the `pcsc-lite` package with the following command:

        ```
        sudo yum install pcsc-lite
        ```
        {: pre}

    2. Install the `libusb` package with the following command:

        ```
        sudo yum install libusb
        ```
        {: pre}

    3. Check and make sure that the `opensc` and `esc` packages are not installed. These packages can cause unexpected errors to occur during the operations of the smart card readers.

        Run the following command to display all installed `opensc` or `esc` packages. Replace `<package_name>` with `opensc` or `esc`:

        ```
        sudo yum list installed <package_name>
        ```
        {: pre}

        If any packages are listed, remove them using the following commands. Replace `<package_name>` with `opensc` or `esc`:

        ```
        sudo yum remove <package_name>
        ```
        {: pre}

    4. Run the `install.sh` script, which is included in the downloaded driver package.

    5. Start the `pcsc` daemon with the following command:

        ```
        sudo pcscd
        ```
        {: pre}



## Step 3: Install the Management Utilities
{: #install-management-utility-application}

Two applications are provided as part of the Management Utilities: the [Smart Card Utility Program](/docs/hs-crypto?topic=hs-crypto-understand-concepts#smart-card-utility-concept) and the [Trusted Key Entry (TKE) application](/docs/hs-crypto?topic=hs-crypto-understand-concepts#tke-client-concept). With the Smart Card Utility Program, you can initialize the smart cards to use. The TKE application uses the smart cards to load master keys in service instances.



To install the applications on Red Hat Enterprise Linux&reg; 8.0.0, complete the following steps:

1. Download the latest installation file, `cloudtke.bin`, from [GitHub](https://github.com/IBM-Cloud/hpcs-management-utilities/releases){: external} to your workstation.
2. (Optional) For maximum security, verify the integrity and authenticity of the Management Utilities installation file `cloudtke.bin` before you install or update the applications.

    {{site.data.keyword.hscrypto}} Management Utilities enable [signed code verification](https://en.wikipedia.org/wiki/Code_signing){: external} to ensure that the signature matches the original code. If the downloaded installation file is altered or corrupted, a different signature is produced and the verification fails. To make sure that the applications are not tampered with or corrupted during the download process, complete the following steps by using the [OpenSSL command-line tool](https://wiki.openssl.org/index.php/Binaries){: external}:

    1. Download the latest version of following files from [GitHub](https://github.com/IBM-Cloud/hpcs-management-utilities/releases){: external} to the same directory where you store the `cloudtke.bin` file:
        - `cloudtke.sig`: The signed cryptographic hash of `cloudtke.bin` (SHA-256).
        - `digicert_cert.pem`: An intermediate code signing certificate to prove the Management Utilities signing certificate.
        - `signing_cert.pem`: The signing certificate of the Management Utilities.

    2. Extract the public key from the signing certificate `signing_cert.pem` to the `sigkey.pub` file with the following command by using the OpenSSL command-line tool:

        ```
        openssl x509 -pubkey -noout -in signing_cert.pem -out sigkey.pub
        ```
        {: pre}

    3. Verify the integrity of the `cloudtke.bin` file with the following command:

        ```
        openssl dgst -sha256 -verify sigkey.pub -signature cloudtke.sig cloudtke.bin
        ```
        {: pre}

        When the verification is successful, `Verified OK` is displayed.

    4. Verify the authenticity and validity of the signing certificate with the following command:

        ```
        openssl ocsp -no_nonce -issuer digicert_cert.pem -cert signing_cert.pem -VAfile digicert_cert.pem -text -url http://ocsp.digicert.com -respout ocsptest
        ```
        {: pre}

        When the verification is successful, `Response verify OK` and `signing_cert.pem: good` are displayed in the output.

    5. If the verification fails, cancel the installation and [contact IBM for support](/docs/hs-crypto?topic=hs-crypto-getting-help). Otherwise, proceed with the following steps.

3. From the command line, enter the directory that the downloaded installation file is located, and perform the following steps to install the applications:

    1. Add the execute permission to the installation file by running the following command:

        ```
        chmod +x cloudtke.bin
        ```
        {: pre}

    2. Install the applications by running the following command:

        ```
        ./cloudtke.bin
        ```
        {: pre}

    3. When prompted, choose the installation folder for the Management Utilities and then select the **Don't create links** option.

## Step 4: Configure smart cards with the Smart Card Utility Program
{: #configure-smart-card-utility}

Two types of smart cards are used in the Management Utilities, the certificate authority smart card and EP11 smart cards. The certificate authority smart card is used to define a smart card zone and create a set of EP11 smart cards in the zone, while an EP11 smart card is used to generate and store an administrator signature key and master key parts.

To configure smart cards, use the Smart Card Utility Program to complete the following tasks.

### Initializing the certificate authority smart card
{: #initialize-ca-smart-card}

To create and initialize a certificate authority smart card, follow these steps:

1. Plug the two smart card readers into the USB ports of your workstation.
2. To start the Smart Card Utility Program, go to the subdirectory where you install the Management Utilities applications and run the following command:

    ```
    ./scup
    ```
    {: pre}

3. After the Smart Card Utility Program is started, from the **CA Smart Card** menu,  select **Initialize and personalize CA smart card**.
4. Insert the smart card to be initialized into a smart card reader 1, and click **OK**.

    If you receive a message similar to **No smart card inserted**, try the following solutions:
    - Insert the smart card into another smart card reader. The smart card reader that you picked might not be smart card reader 1.
    - Reinsert the smart card into the same smart card reader. The smart card reader might not be reading the smart card properly.

5. Enter a six-digit personal identification number (PIN) twice on the smart card reader PIN pad as the first certificate authority card PIN.

    Enter the PIN twice in 20 seconds. Otherwise, the session is expired and you need to reenter the PIN.
    {: tip}

6. Enter a six-digit PIN twice on the smart card reader PIN pad as the second certificate authority card PIN. You need to enter both PINs later when you use the certificate authority smart card.

    Enter the PIN twice in 20 seconds. Otherwise, the session is expired and you need to reenter the PIN.
    {: tip}

7. Enter a description for the smart card zone that holds the EP11 smart cards.
8. Enter a description for the certificate authority smart card.

The certificate authority is created.

It is suggested to create a backup copy of your certificate authority smart card. You can create a backup copy by clicking **CA Smart Card** &gt; **Backup CA smart card**.
{: important}

### Initializing the EP11 smart card
{: #initialize-ep11-smart-card}

To create and initialize an EP11 smart card, follow these steps:

1. Select **EP11 Smart Card** &gt; **Initialize and enroll EP11 smart card** from the menu.
2. Insert the smart card to be initialized as the EP11 smart card in smart card reader 2 and click **OK**.
3. Select **EP11 Smart Card** &gt; **Personalize EP11 smart card** from the menu bar.
4. Enter a six-digit PIN twice on the smart card reader PIN pad. You need to enter this PIN later when you use the EP11 smart card.
5. Enter a description for the EP11 smart card.
6. If you have more than one EP11 smart card, repeat step 1 to step 5 to initialize other EP11 smart cards.

Your EP11 smart cards are created and personalized.



## What's next
{: #prepare-management-utilities-next}

- After you set up the Management Utilities, you can start to load the master key with the TKE application. For detailed instructions, see [Initializing service instances using smart cards and the Management Utilities](/docs/hs-crypto?topic=hs-crypto-initialize-hsm-management-utilities).
- If you need to uninstall the Management Utilities, you need to follow the instructions to [zeroize your crypto units first](/docs/hs-crypto?topic=hs-crypto-delete-instance#zeroize-crypto-unit-step) and then [uninstall the Management Utilities](/docs/hs-crypto?topic=hs-crypto-delete-instance#uninstall-management-utilities).
