---

copyright:
  years: 2020, 2021
lastupdated: "2021-08-09"

keywords: troubleshoot, problems, known issues, unauthorized when running TKE CLI plug-in commands

subcollection: hs-crypto

---

{:shortdesc: .shortdesc}
{:codeblock: .codeblock}
{:screen: .screen}
{:tsSymptoms: .tsSymptoms}
{:tsCauses: .tsCauses}
{:tsResolve: .tsResolve}
{:pre: .pre}
{:tip: .tip}
{:codeblock: .codeblock}
{:troubleshoot: data-hd-content-type='troubleshoot'}
{:external: target="_blank" .external}
{:support: data-reuse='support'}
{:term: .term}

# Why am I not authorized when running TKE CLI plug-in commands?
{: #troubleshoot-unauthorized-token}
{: troubleshoot}
{: support}

You receive an error message after you run a `tke` CLI command.
{: shortdesc}

The message might be similar to the following one:
{: tsSymptoms}

```
ibmcloud tke cryptounits
FAILED
Error querying service instances.
Status code: 401
Message: Unauthorized
Your access token is invalid, expired, or does not have the necessary permissions to access this instance.
```
{: codeblock}

To run TKE CLI plug-in commands that send requests to the {{site.data.keyword.cloud_notm}}, you must have a valid authentication token. An authentication token is created when you log in to the {{site.data.keyword.cloud_notm}}, but it expires after 1 hour.  After 1 hour, you must log in again to continue to send requests to the {{site.data.keyword.cloud_notm}}.
{: tsCauses}

Log in to {{site.data.keyword.cloud_notm}} again with the `ibmcloud login` command to refresh the token.
{: tsResolve}
