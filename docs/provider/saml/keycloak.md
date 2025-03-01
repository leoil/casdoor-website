---
title: Keycloak
description: Using Keycloak to authenticate users
keywords: [Keycloak]
authors: [seriouszyx]
---

The JBoss [KeyCloak](https://www.keycloak.org/) system is a widely used and open-source identity management system that supports integration with applications via SAML and OpenID Connect. It also can operate as an identity broker between other providers such as LDAP or other SAML providers and applications that support SAML or OpenID Connect.

The following is an example of how to configure a new client entry in KeyCloak and configure Casdoor to use it to permit UI login by KeyCloak users that are granted access via KeyCloak configuration.

## Configure Keycloak

Some config choices and assumptions specifically for this example:

- Let’s assume that you are running Casdoor as dev mode locally. Casdoor UI is available at: `http://localhost:7001` and server is available at `http://localhost:8000`. Replace with the appropriate url as needed.
- Let's assume that you are running Keycloak locally. Keycloak UI is available at: `http://localhost:8080/auth`.
- Based on that, the SP ACS URL for this deployment will be: `http://localhost:8000/api/acs`.
- Our SP Entity ID will use the same url: `http://localhost:8000/api/acs`.

Use the default realm or create a new realm.

![Add keycloak realm](/img/providers/SAML/keycloak_realm_add.png)

![Keycloak realm](/img/providers/SAML/keycloak_realm.png)

## Add a client entry in Keycloak

:::info

See more details about Keycloak Clients in [Keycloak documentation](https://www.keycloak.org/docs/latest/server_admin/index.html#_client-saml-configuration).

:::

Click **Clients** in the menu and then click **Create** to go to the **Add Client** page. Fill in as below.

- **Client ID**: `http://localhost:8000/api/acs` - This will be the SP Entity ID used in the Casdoor configuration later.
- **Client Protocol**: `saml`.
- **Client SAML Endpoint**: `http://localhost:8000/api/acs`. - This URL is where you want the Keycloak server to send SAML requests and responses. Generally, applications have one URL for processing SAML requests. Multiple URLs can be set in the Settings tab of the client.

![Add keycloak client](/img/providers/SAML/keycloak_client_add.png)

Click **Save**. This action creates the client and brings you to the **Settings** tab.

The following list part of settings:

1. **Name** - `Casdoor`. This is only used to display a friendly name to Keycloak users in the KeyCloak UI. You can use any name you like.
2. **Enabled** - Select on.
3. **Include Authn Statement** - Select on.
4. **Sign Documents** - Select on.
5. **Sign Assertions** - Select off.
6. **Encrypt Assertions** - Select off.
7. **Client Signature Required** - Select off.
8. **Force Name ID Format** - Select on.
9. **Name ID Format** - Select username.
10. **Valid Redirect URIs** - Add `http://localhost:8000/api/acs`.
11. **Master SAML Processing URL** - `http://localhost:8000/api/acs`.
12. Fine Grain SAML Endpoint Configuration
    1. **Assertion Consumer Service POST Binding URL** - `http://localhost:8000/api/acs`.
    2. **Assertion Consumer Service Redirect Binding URL** - `http://localhost:8000/api/acs`.

Save the configuration.

![Configure keycloak client](/img/providers/SAML/keycloak_client_configure.png)

:::tip

If you want to sign authn request, you need to enable the **Client Signature Required** option and upload the certificate generated by yourself. The private key and certificate using in Casdoor, `token_jwt_key.key` and `token_jwt_key.pem` are located in the **object** directory. In Keycloak, you need to click the **Keys** tab, click **Import** button, select **Archive Format** as **Certificate PEM** and upload the certificate.

:::

Click **Installation** tab.

For the Keycloak <= 5.0.0, select Format Option - **SAML Metadata IDPSSODescriptor** and copy the metadata.

For Keycloak 6.0.0+, select Format Option - **Mod Auth Mellon files** and click **Download**. Unzip the downloaded.zip, locate `idp-metadata.xml` and copy the metadata.

![Download metadata](/img/providers/SAML/keycloak_client_install.png)

![Copy metadata](/img/providers/SAML/keycloak_client_copy.png)

## Configure in Casdoor

Create a new provider in Casdoor.

Select category as **SAML**, type as **Keycloak**. Copy the content of metadata and paste it to the **Metadata** input. The values of **Endpoint**, **IdP** and **Issuer URL** will be generated automatically after clicking the **Parse** button. Finally click the button **Save**.

:::tip

If you enable the **Client Signature Required** option in Keycloak and upload the certificate, please enable the **Sign request** option in Casdoor.

:::

![Casdoor provider](/img/providers/SAML/keycloak_casdoor_provider.png)

Edit the application you want to configure in Casdoor. Select the provider just added and click the button **Save**.

![Add provider for app](/img/providers/SAML/keycloak_casdoor_app.png)

## Validate the effect

Go to the application you just configured and you can find that there is a Keycloak icon in the login page.

Click the icon and jump to the Keycloak login page, and then successfully login to the Casdoor after authentication.

![Casdoor login](/img/providers/SAML/keycloak_casdoor_login.gif)
