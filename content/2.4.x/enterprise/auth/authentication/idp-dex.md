---
# metadata # 
title: Identity Providers
description: Learn how to enable users to log in to Pachyderm using their preferred identity provider. 
date: 
# taxonomy #
tags: ["identity-providers", "permissions", "management", "integrations"]
series:
seriesPart:
---

{{% notice note %}}
- Return to our [Enterprise landing page](https://docs.pachyderm.com/latest/enterprise/) if you do not have an enterprise key.
- Helm users, **you have the option to set your IdP values directly through Helm (Recommended)**. See below.
- Alternatively, you can use `pachctl` to connect your IdP to Pachyderm. First, verify that the [Authentication](../../#authentication-and-authorization) is enabled by running `pachctl auth whoami`. The command should return `You are "pach:root" `(i.e., your are the **Root User** with `clusterAdmin` privileges).  Run `pachctl auth use-auth-token` and enter your rootToken to login as a Root User if you are not.
{{% /notice %}}
    
{{% notice attention%}}
We are now shipping Pachyderm with an **optional embedded proxy** 
allowing your cluster to expose one single port externally. This deployment setup is optional.

If you choose to deploy Pachyderm with a Proxy:

- Check out our new recommended architecture and [deployment instructions](../../../../deploy-manage/deploy/deploy-w-proxy/).
- **Update your Callback URL**. Details below.
{{% /notice %}}
    

## Enable your users to authenticate to Pachyderm by logging into their favorite Identity Provider in 3 steps:

1. [Register the Pachyderm Application with your IdP](#1-register-a-pachyderm-application-with-your-idp).
1. [Set up and create your Idp-Pachyderm connector](#2-set-up-and-create-an-idp-pachyderm-connector).
1. [Login](#3-login).

Your users should now be able to [login to Pachyderm](../login).

We chose to illustrate those steps
by using Auth0 as our Identity Provider.
([Auth0](https://auth0.com/) is an open source, online authentication platform that
users can use to log in to various applications).

However, Pachyderm's Identity Service is based on [Dex](https://dexidp.io/docs/)
and can therefore provide connectors to a large [variety of IdPs](https://dexidp.io/docs/connectors/) (LDAP, GitHub, SAML, OIDC...). 
Use the IdP of your choice.



For now, let's configure Pachyderm so that our
Pachyderm users can log in through Auth0.

### 1: Register a Pachyderm Application with your IdP

{{% notice tip %}}
The one important and invariant element of this step, 
no matter what your IdP choice might be, is the **callback URL**.
Callback URLs are the URLs that your IdP invokes after the authentication process. 
The IdP redirects back to this URL once a user is authenticated.

For security reasons, you need to add your application's URL to your client's Allowed Callback URLs.
This enables your IdP to recognize these URLs as valid. 

**For Local or “Quick” deployment cases where you do not have a public DNS entry or public IP address, set the following field `config.insecureSkipIssuerCallbackDomainCheck` to `true` in your connector file below.**

The format of the URL is described below. 
{{% /notice %}}

If you do not have an Auth0 account, sign up for one
at https://auth0.com and create your Pool of Users 
(although this step might be done later).

Then, complete the following steps:

1. Log in to your Auth0 account.
1. In **Applications**, click **Create Application**.
1. Type the name of your application, such as **Pachyderm**.
1. In the application type, select **Regular Web Application**.
1. Click **Create**.
1. Go to the application settings.
1. Scroll down to **Application URIs**.
1. In the **Allowed Callback URLs**, add the Pachyderm callback link in the
   following format:

    ```s
    # Dex's issuer URL + "/callback"
    http://<insert-external-ip-or-dns-name>:30658/callback
    ```

    The IP address is the address of your Pachyderm host. For example,
    if you are running Pachyderm in Minikube, you can find the IP
    address by running `minikube ip`. 

   {{% notice warning %}} 
   **Attention Proxy users**: 
    Your Callback URL must be set to `http(s)://<insert-external-ip-or-dns-name>/dex/callback`. 
    {{% /notice %}}

1. Scroll down to **Show Advanced Settings**.
1. Select **Grant Types**.
1. Verify that **Authorization Code** and **Refresh Token** are selected.

   ![Auth0 Grant Settings](/images/auth0-grant-settings.png)

{{% notice note %}}
For this Auth0 example, we have created a user in Auth0 in **User Management/Users**.
We will log in to Pachyderm as this user once our IdP connection is completed.
![Auth0 Create User](/images/auth0-create-user.png)
{{% /notice %}}

### 2: Set up and create an Idp-Pachyderm connector

#### Create A Connector Configuration File
To configure your Idp-Pachyderm integration, **create a connector configuration file** corresponding to your IdP. 

{{% notice info %}}
For a list of available connectors and their configuration options, see [Dex documentation](https://dexidp.io/docs/connectors/).
{{% /notice%}}

In the case of our integration with Auth0, we will use an oidc connector with the following parameters:

{{% notice note %}}
Pachyderm supports the JSON and YAML formats for its connector files. 
{{% /notice %}}

See our oidc connector example in JSON and YAML formats below.

#####  oidc-dex-connector.json

``` json
{
"type": "oidc",
"id": "auth0",
"name": "Auth0",
"version": 1,
"config":{
  "issuer": "https://dev-k34x5yjn.us.auth0.com/",
  "clientID": "hegmOc5rTotLPu5ByRDXOvBAzgs3wuw5",
  "clientSecret": "7xk8O71Uhp5T-bJp_aP2Squwlh4zZTJs65URPma-2UT7n1iigDaMUD9ArhUR-2aL",
  "redirectURI": "http://<ip>:30658/callback",
  "insecureEnableGroups": true,
  "insecureSkipEmailVerified": true,
  "insecureSkipIssuerCallbackDomainCheck": false,
  "forwardedLoginParams": ["login_hint"] 
  }
}
```
##### oidc-dex-connector.yaml

``` yaml
  type: oidc
  id: auth0
  name: Auth0
  version: 1
  config:
      issuer: https://dev-k34x5yjn.us.auth0.com/
      clientID: hegmOc5rTotLPu5ByRDXOvBAzgs3wuw5
      clientSecret: 7xk8O71Uhp5T-bJp_aP2Squwlh4zZTJs65URPma-2UT7n1iigDaMUD9ArhUR-2aL
      redirectURI: http://<ip>:30658/callback
      insecureEnableGroups: true
      insecureSkipEmailVerified: true
      insecureSkipIssuerCallbackDomainCheck: false,
      forwardedLoginParams:
      - login_hint
```

You will need to replace the following placeholders with relevant values:

- `id`: The unique identifier of your connector (string).

- `name`: Its full name (string).

- `type`: The type of connector. (oidc, saml).

- `version`:The version of your connector (integer - default to 0 when creating a new connector)

- `issuer` — The domain of your application (here in Auth0). For example,
`https://dev-k34x5yjn.us.auth0.com/`. **Note the trailing slash**.

- `client_id` — The Pachyderm **Client ID** (here in Auth0). The client ID
consists of alphanumeric characters and can be found on the application
settings page.

- `client_secret` - The Pachyderm client secret (here in Auth0) located
on the application settings page.

- `redirect_uri` - This parameter should match what you have added
to **Allowed Callback URLs** when registering Pachyderm on your IdP website.

{{% notice warning %}}
**When using an [ingress](../../../../deploy-manage/deploy/ingress/#ingress)**:

- `redirect_uri` must be changed to point to `https://domain-name/dex/callback`. (Note the additional **/dex/**) 
- TLS requires all non-localhost redirectURIs to be **HTTPS**.
- AZURE USERS: 
    - You must use TLS when deploying on Azure.
    - When using Azure Active Directory, add the following to the oidc config:
    ``` yaml
    "config":{
        "claimMapping": {
            "email": "preferred_username"
        } 
    }      
 
    ```
{{% /notice %}}

{{% notice warning %}} 
Attention Proxy users
Your `redirect_uri` must be set to `http(s)://<insert-external-ip-or-dns-name>/dex/callback`. 
{{%/notice %}}

{{% notice note %}}
Note that Pachyderm's YAML format is **a simplified version** of Dex's [sample config](https://dexidp.io/docs/connectors/oidc/).
{{% /notice %}}

#### Create Your Idp-Pachyderm Connection

Once your Pachyderm application is registered with your IdP (here Auth0), 
and your IdP-Pachyderm connector config file created (here with the Auth0 parameters), **connect your IdP to Pachyderm** in your Helm values (recommended) or by using `pachctl`:

- Reference your connector in Helm

    Provide your connector info in the [oidc.upstreamIDPs](https://github.com/pachyderm/pachyderm/blob/{{% majorMinorVersion %}}/etc/helm/pachyderm/values.yaml#L774) field of your helm values. Pachyderm will store this value in the platform secret `pachyderm-identity` in the key upstream-idps.

    Alternatively, you can [create a secret](../../../../how-tos/advanced-data-operations/secrets/#generate-your-secret-configuration-file) containing your dex connectors (Key: upstream-idps) and reference its name in the field [oidc.upstreamIDPsSecretName](https://github.com/pachyderm/pachyderm/blob/{{% majorMinorVersion %}}/etc/helm/pachyderm/values.yaml#L805).

##### Example 
Below, a yaml example of the `stringData` section of an IdP generic secret.

```yaml
stringData:
upstream-idps: |
    - type: github
    id: github
    name: GitHub
    jsonConfig: >-
        {
        "clientID": "xxx",
        "clientSecret": "xxx",
        "redirectURI": "https://pach.pachdemo.cloud/dex/callback",
        "loadAllGroups": true
        }
```


- Alternatively, use `pachctl`

```s
pachctl idp create-connector --config oidc-dex-connector.json
```
or
```s
pachctl idp create-connector --config oidc-dex-connector.yaml
```
Check your connector's parameters by running:
```s
pachctl idp get-connector <your connector id: auth0>
```

Per default, the `version` field of the connector is set to 0 when created.
However, you can set its value to a different integer.

You will specifically need to increment this value when updating your connector.
```s
pachctl idp update-connector <your connector id: auth0> --version 1
```
or
```s
pachctl idp update-connector --config oidc-dex-connector.yaml
```
{{% notice info %}}
Run `pachctl idp --help` for a full list of commands. In particular, those commands let you create, update, delete, list, or get a specific connector.
{{%/notice%}}

### 3- Login
The users registered with your IdP are now ready to [Log in to Pachyderm](../login)

## User Revocation

Use the `pachctl auth revoke` command to revoke access for an existing Pachyderm user (for example, a robot user accessing your cluster, a team member leaving, etc... ). In particular, you can:

- revoke a given token: `pachctl auth revoke --token=<pach token>`.
- revoke all tokens for a given user `pachctl auth revoke --user=idp:usernamen@pachyderm.io` to log that user out forcibly.

{{% notice note %}}
Note that a user whose Pachyderm token has been revoked can technically log in to Pachyderm again unless **you have removed that user from the user registry of your IdP**.
{{% /notice %}}

For the curious mind: Take a look at the sequence diagram below illustrating the OIDC login flow. It highlights the exchange of the original OIDC ID Token for a Pachyderm Token.

![OIDC Login Flow](/images/pachyderm-oidc-dex-flow.png)

