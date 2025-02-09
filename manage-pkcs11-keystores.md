---

copyright:
  years: 2020, 2021
lastupdated: "2021-08-19"

keywords: pkcs11 keystore, ep11 keystore, create pkcs11 keystore, generate pkcs11 keystore, view ep11 keystore, delete ep11 keystore, view pkcs11 keystore

subcollection: hs-crypto

---


{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}
{:important: .important}
{:note: .note}
{:tip: .tip}
{:external: target="_blank" .external}
{:term: .term}
{:help: data-hd-content-type='help'}
{:support: data-reuse='support'}

# Managing EP11 keystores with the {{site.data.keyword.cloud_notm}} console
{: #manage-ep11-keystores-ui}

Enterprise PKCS #11 (EP11) keystores are used to store EP11 keys. Before you create EP11 keys, you need to create EP11 keystores first. Apart from using the {{site.data.keyword.hscrypto}} [PKCS #11 API](/docs/hs-crypto?topic=hs-crypto-set-up-pkcs-api) to manage EP11 keystores, you can also use the {{site.data.keyword.cloud}} console to view, create, and delete EP11 keystores.
{: shortdesc}

EP11 keystores are composed of in-memory keystores and database-backed keystores. The in-memory EP11 keystores are not displayed in the {{site.data.keyword.cloud_notm}} console. You can manage only database-backed EP11 keystores through the console. For more information about the keystores, see [Introducing keystore](/docs/hs-crypto?topic=hs-crypto-pkcs11-intro#pkcs11-keystore-intro).
{: important}

## Before you begin
{: #manage-ep11-keystores-ui-before}

Before you can manage EP11 keystores with the console, complete the following steps to open your {{site.data.keyword.hscrypto}} dashboard:

1. [Log in to the {{site.data.keyword.cloud_notm}} console](https://cloud.ibm.com/login){: external}.
2. Go to **Menu** &gt; **Resource List** to view a list of your resources.
3. From your {{site.data.keyword.cloud_notm}} resource list, select your provisioned instance of {{site.data.keyword.hscrypto}} and click the instance name.

## Viewing EP11 keystores
{: #view-ep11-keystore-ui}

For default service access roles that support viewing EP11 keystores, see [service access roles](/docs/hs-crypto?topic=hs-crypto-manage-access#service-access-roles). If you are to create custom roles, make sure to assign the following action to the custom role:

* `hs-crypto.keystore.listkeystoresbyids`

For instructions on creating custom roles, see [Creating custom roles](/docs/account?topic=account-custom-roles).

To view a list of created EP11 keystores, on the {{site.data.keyword.hscrypto}} dashboard, select the **EP11 keystores** tab in the side menu.

An EP11 keystore table is displayed with the following details.

<table>
<tr>
    <th>Setting</th>
    <th>Description</th>
</tr>
<tr>
    <td>ID</td>
    <td>
    <p>The unique identifier that is assigned when the keystore is created.</p>
    </td>
</tr>
<tr>
    <td>Name</td>
    <td>
    <p>A human-readable alias for easy identification of your keystore.</p>
    <p>The keystore name might not be unique. You can assign multiple keystores with the same name. If there is no name associated with the keystore, it means the keystore is created using the PKCS #11 API.</p>
    </td>
</tr>
<tr>
    <td>Type</td>
    <td>
    <p>The type of the keystore. Possible values are <strong>Public</strong> and <strong>Private</strong>.</p>
    <p>The database-backed EP11 keystores are composed of two types of keystores: public keystores for storing less sensitive EP11 keys that can be accessed by any user types (security officers, normal users, and anonymous users) and private keystores for storing sensitive EP11 keys that can be accessed by normal users only. For more information about keystores, see <a href="/docs/hs-crypto?topic=hs-crypto-pkcs11-intro#pkcs11-keystore-intro">Introducing keystore</a>.</p>
    </td>
</tr>
    <caption>Table 1. Describes the EP11 keystores table</caption>
</table>

## Creating EP11 keystores
{: #create-ep11-keystore-ui}
{: help}
{: support}

For default service access roles that support creating EP11 keystores, see [service access roles](/docs/hs-crypto?topic=hs-crypto-manage-access#service-access-roles). If you are to create custom roles, make sure to assign the following actions to the custom role:

* `hs-crypto.keystore.listkeystoresbyids`
* `hs-crypto.keystore.createkeystore`

For instructions on creating custom roles, see [Creating custom roles](/docs/account?topic=account-custom-roles).

Complete the following steps to create an EP11 keystore:

1. Select the **EP11 keystores** tab in the side menu. You can see a list of existing EP11 keystores with each having a unique ID.
2. Click **Create keystore**. In the **Create EP11 keystore** panel that is displayed, enter the **Keystore name** and select the **Keystore type** to be **Private** or **Public**.

    The database-backed EP11 keystores are composed of two types of keystores: public keystores for storing less sensitive EP11 keys that can be accessed by any user types (security officers, normal users, and anonymous users) and private keystores for storing sensitive EP11 keys that can be accessed by normal users only. For more information about keystores, see [Introducing keystore](/docs/hs-crypto?topic=hs-crypto-pkcs11-intro#pkcs11-keystore-intro).

3. Click **Create keystore**. You can see the new keystore listed at the first row.

    To copy the keystore ID, click the **Actions** icon ![Actions icon](../icons/action-menu-icon.svg "Actions") and then click **Copy ID to clipboard**.
    {: tip}

## Deleting EP11 keystores
{: #delete-ep11-keystore-ui}

For default service access roles that support deleting EP11 keystores, see [service access roles](/docs/hs-crypto?topic=hs-crypto-manage-access#service-access-roles). If you are to create custom roles, make sure to assign the following actions to the custom role:

* `hs-crypto.keystore.listkeystoresbyids`
* `hs-crypto.keystore.deletekeystore`

For instructions on creating custom roles, see [Creating custom roles](/docs/account?topic=account-custom-roles).

If you want to delete an EP11 keystore, complete the following steps:

After you delete a keystore, you are not able to access any EP11 keys that are stored in the keystore. This action cannot be undone.
{: important}

1. Select the **EP11 keystores** tab in the side menu, and find the keystore that you want to delete in the list.
2. Click the **Actions** icon ![Actions icon](../icons/action-menu-icon.svg "Actions"), and then select **Delete keystore**.
3. Verify the ID of the keystore to be deleted, and check the box to confirm the deletion.
4. Click **Delete keystore**.

## What's next
{: #manage-ep11-keystore-ui-next}

Start to create an EP11 key [through the console UI](/docs/hs-crypto?topic=hs-crypto-create-ep11-key-ui) or [through the PKCS #11 API](/docs/hs-crypto?topic=hs-crypto-set-up-pkcs-api).
