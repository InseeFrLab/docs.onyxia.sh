---
icon: key-skeleton
---

# OpenID Connect Configuration

The instalation guides instruct you on how to instantiate a new Keycloak to enable authentication on your datalab.

However chance are your organization already have an existing IAM system in place in your organization. This guide covers how to integrate Onyxia with various commonly used OIDC providers, namely [Keycloak](https://www.keycloak.org/), [Auth0](https://auth0.com/) and [Microsoft Entra ID](https://www.microsoft.com/en-us/security/business/identity-access/microsoft-entra-id). &#x20;

## Overview of the avalible parameters

Before diving into specifc OIDC providers, review the available parameters. &#x20;

{% code title="apps/onyxia/values.yaml" %}
```yaml
onyxia:
  api:
    env:
      # Mandatory and no other mode is currently supported.
      authentication.mode: "openidconnect"

      # Mandatory: The issuer URI of the OIDC provider.  
      oidc.issuer-uri: "..."

      # Mandatory: The client ID of the OIDC client that represents the Onyxia Web Application.
      oidc.clientID: "..."

      # Mandatory: Defines which claim in the Access Token's JWT serves as the unique 
      # user identifier.  
      # This identifier must contain only lowercase alphanumeric characters and `-`. 
      # More specificaly it must complies with the RFC 1123: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#dns-label-names
      #
      # - If your usernames conform to this constraint, you can use 
      #   `"preferred_username"` for a more human-readable identifier.
      # - If your usernames may contain special characters, use another claim 
      #   like `"sub"` (Make sure the `sub` values actually complies with RFC 1123).  
      #
      oidc.username-claim: "..."

      # Optional: Defaults to `"groups"`. Defines which claim represents user groups.
      oidc.groups-claim: "..."

      # Optional: Defaults to `"roles"`. Defines which claim represents user roles.
      oidc.roles-claim: "..."

      # Optional: Additional query parameters to append to the OIDC provider login URL.  
      # Example: If using Keycloak with Google OAuth as an identity provider, you might want  
      # to preselect Google as the login option using `"kc_idp_hint=google"`.  
      # 
      # ⚠️ This string is appended as-is. Ensure it's properly URI-encoded.  
      # If adding multiple parameters, separate them with `&`.  
      #
      # Example: `"foo=foo%20value&bar=bar%20value"`
      oidc.extra-query-params: "..."

      # Optional: Specifies the expected audience value in the Access Token.  
      # If provided, Onyxia will validate the `aud` claim in the token and reject 
      # requests where it does not match (or include an entries that match if it's 
      # an array).
      oidc.audience: "..."

      # Optional: Specifies the OIDC scopes requested by the Onyxia client.  
      # Defaults to `"openid profile"`.  
      # This is a space-separated list. `"openid"` is always requested, 
      # regardless of this setting.
      oidc.scope: "..."
      
      # Optional: This parameter is to be provided if you want your user to be
      # automatically logged out after a set period of inactivity.  
      oidc.idleSessionLifetimeInSeconds: "..."

      # Optional: Onyxia API fetches `<issuer-uri>/.well-known/openid-configuration` 
      # to retrieve JWKs  for validating Access Tokens (used as Authorization Bearers).  
      #
      # ⚠️ In development, if you lack proper root certificates, you can disable TLS verification.  
      # However, in production, it's recommended to mount the correct `cacerts` instead.
      oidc.skip-tls-verify: "true|false"
      

```
{% endcode %}

***

## OIDC Provider Specific Configuration Guides

{% tabs %}
{% tab title="Keycloak" %}
### Onyxia Login Theme

We ship [a custom Login Keycloak theme](https://youtu.be/NrVuVXsbloA?si=fDCPpXUIpSlCHsYw\&t=405) with each version of Onyxia. You can dowload it [on the relase page](https://github.com/InseeFrLab/onyxia/releases). You will find specific instruction on how to load the theme in your onyxia instance [in this guide](https://docs.keycloakify.dev/deploying-your-theme). If you are using Helm for deploying Keycloak as instructed in the instalation guide[ here are the relevent lines](https://github.com/InseeFrLab/onyxia-ops/blob/35f86c848a3ddeef6bfe4a9a4f41e5d516eb66db/apps/keycloak/values.yaml#L60-L79) in the Onyxia-ops repo.

### Deciding wich claim of the Access Token to use as unique user identifier

Onyxia needs to have an unique identifier for your users. To configure this you tell onyxia in wich claim of the Access Token issued by your provider onyxia should find the unique identifier. &#x20;

It is nice to be able to use the `preffered_usename` as identifier however if you want to do so you must ensure that it matches the [RFC 1123](https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#dns-label-names) that is to say that it contain only lowercase alphanumerical character and -.\
\
This is a pretty restrictive format. If you already have an existing userbase you can't use `preferred_username`, so you have two option, either configure a custom mapper in your keycloak instance so that a complying claim is generated on your access token, or simply use `"sub"`.

On the other hand, if you are starting with no existing user you can define a rexept on the User Profile Attributes of "username" so that users are forced to pick a username that comply with the restrictions.

There are info on how to do so in [the installation guide](https://docs.onyxia.sh/admin-doc/readme/user-authentication) (search for the word "pattern").

### Getting specific instuction on how to configure Keycloak.

Beyond what's explained in the installation guide, if you want more generic instruction on how to configure a Keycloak public Keycloak OIDC client like Onyxia you can refer to the following guide, it comes with a simple project to let you test your configuration.

This guide is generic, in the context of Onyxia you here are the sugested substitutions:

* **\<KC\_DOMAIN>**: `auth.lab.my-domain.net`
* **\<KC\_RELATIVE\_PATH>:** `/auth`
* **\<REALM\_NAME>**: `datalab`
* **\<APP\_DOMAIN>:** `datalab.my-domain.net`
* **\<BASE\_URL>**: `/`
* **\<DEV\_PORT>**: `5173`

{% embed url="https://docs.oidc-spa.dev/providers-configuration/keycloak" %}

Here is an overview of what your onyxia values.yaml should look like:

{% code title="apps/onyxia/values.yaml" %}
```yaml
onyxia:
  api:
    env:
      authentication.mode: "openidconnect"
      # Example: "https://auth.lab.my-domain.net/auth/realms/datalab"
      oidc.issuer-uri: "https://<KC_DOMAIN><KC_RELATIVE_PATH>/realms/<REALM_NAME>"
      # Example: "onyxia"
      oidc.clientID: "<ONYXIA_CLIENT_ID>"
      # Examples:
      # `"preferred_username"` if you have configured a regexp for the username
      # `"my-custom-claim"`    if you have setup a custom mapper.
      # `"sub"`                will always work
      oidc.username-claim: "..."
```
{% endcode %}
{% endtab %}

{% tab title="Microsoft Entra ID" %}
Follow the following guide for instruction on how to configure an Entra ID application for Onyxia.

This guide is generic, in the context of Onyxia, here are the recommended substitution:

* "My App" -> "Onyxia"
* "My App - API" -> "Onyxia - API"
* "api://my-app-api" -> "api://onyxia-api"
* "https://my-app.com/" -> "https://datalab.my-domain.net/"

{% embed url="https://docs.oidc-spa.dev/providers-configuration/microsoft-entra-id" %}

Here is what your configuration should look like:

{% code title="apps/onyxia/values.yaml" %}
```yaml
onyxia:
  api:
    env:
      authentication.mode: "openidconnect"
      oidc.issuer-uri: "https://login.microsoftonline.com/<Directory (tenant) ID (Onyxia)>/v2.0"
      oidc.clientID: "<Application (client) ID (Onyxia)>"
      # ⚠️ Do **not** use `"sub"` or `"upn"` since they may contain 
      # non-alphanumeric characters.  
      oidc.username-claim: "oid"
      # Example: "profile api://onyxia-api/access_as_user"
      oidc.scope: "profile <Application ID URI (Onyxia - API)>/<scope name (usually access_as_user)>"
      # Example: "api://onyxia-api"
      onyxia.audience: "<Application ID URI (Onyxia - API)>"
```
{% endcode %}
{% endtab %}

{% tab title="Auth0" %}
Follow the following guide for instruction on how to configure an Entra ID application for Onyxia.

This guide is generic, in the context of Onyxia, here are the recommended substitution:

* "My App" -> "Onyxia"
* **\<APP\_DOMAIN>:** `datalab.my-domain.net`
* **\<BASE\_URL>**: `/`
* **\<DEV\_PORT>**: `5173`
* "My App - API" -> "Onyxia - API"
* `https://myapp.my-company.com/api` -> `https://datalab.my-domain.net/api`
* auth.my-company.com -> auth.my-domain.net

{% embed url="https://docs.oidc-spa.dev/providers-configuration/microsoft-entra-id" %}

### Generating an RFC 1123 Compliant Claim in the Access Token

By default, Auth0 does not issue any claim that Onyxia can use as a unique user identifier, so you need to create one.\
This can be achieved by defining a **custom claim** in the access token using an Auth0 **Trigger Action**.

#### Steps to Create the `onyxia-username` Claim

Follow these steps to configure Auth0 to include a **RFC 1123-compliant** identifier in your access token.

1️⃣ Create a Custom Action: &#x20;

1. Go to **Auth0 Dashboard** → **Actions** → **Library**.
2. Click **Create Action**.
3. Set the following values:
   * **Name**: `GenerateOnyxiaUsername`
   * **Trigger**: **Post Login**
   * **Runtime**: `Node 22`
4. Click **Create**.

2️⃣ Add the Custom Code

Replace the default content with the following JavaScript code:

```js
function toRFC1123(input) {
  if (!input) return "";
  let output = input.toLowerCase();
  output = output.replace(/[^a-z0-9-]/g, "-");
  output = output.replace(/^-+|-+$/g, "");
  if (output.length > 63) {
    output = output.substring(0, 63);
    output = output.replace(/-+$/, "");
  }
  return output;
}

exports.onExecutePostLogin = async (event, api) => {
  const sub = event.user.user_id;
  if (sub) {
    api.accessToken.setCustomClaim("onyxia-username", toRFC1123(sub));
  }
};
```

3️⃣ Deploy and Activate the Action

1. Click **Deploy**.
2. Navigate to **Auth0 Dashboard** → **Actions** → **Triggers** → **Post Login**.
3. Drag and drop the newly created `GenerateOnyxiaUsername` action into the flow.
4. Click **Apply Changes**.

Now your access token should be generated with an `onyxia-username` claim!

<figure><img src="../.gitbook/assets/image (52).png" alt="" width="375"><figcaption><p>Preview of the decoded JWT of the Access Token issued by Auth0<br>with the custom action enabled when previewed with the<br>test app of the oidc-spa guide</p></figcaption></figure>

### Final configuration

Here is what your configuration should look like:

{% code title="apps/onyxia/values.yaml" %}
```yaml
onyxia:
  api:
    env:
      authentication.mode: "openidconnect"
      oidc.issuer-uri: "https://auth.my-domain.net"
      oidc.clientID: "<Onyxia Application Client ID>"  
      oidc.username-claim: "onyxia-username"
      oidc.audience: "https://datalab.my-domain.net/api"
      # If you have configured Auto Logout:
      oidc.idleSessionLifetimeInSeconds: "300"
```
{% endcode %}
{% endtab %}

{% tab title="Other" %}
If you are using another OIDC provider and you have issue configuring Onyxia to work with it. Please reach out [on Slack](https://join.slack.com/t/3innovation/shared_invite/zt-2skhjkavr-xO~uTRLgoNOCm6ubLpKG7Q) we'll be happy to shedule a call and help with the integration.&#x20;

Here are, however some generic instruction on how to create an OIDC client for Onyxia.

In the context of Onyxia, `https://my-app.com/` is `https://datalab.my-domain.net/`.

{% embed url="https://docs.oidc-spa.dev/providers-configuration/other" %}
{% endtab %}
{% endtabs %}

## **OIDC Configuration for Services Onyxia Connects To**

Onyxia uses an OIDC client for authentication, but it also connects to other OIDC-enabled services.\
Each of these services **can** have its own OIDC configuration, allowing Onyxia to authenticate\
using a separate client identity.

In the **region configuration**, you can specify an optional `oidcConfiguration` object for\
each service:

* **S3 (MinIO STS)** → `onyxia.api.regions[].data.S3.sts.oidcConfiguration`
* **Vault** → `onyxia.api.regions[].vault.oidcConfiguration`
* **Kubernetes API** → `onyxia.api.regions[].services.k8sPublicEndpoint.oidcConfiguration`

Each configuration follows this structure:

```ts
type OidcConfiguration = {
    issuerURI?: string;
    clientID?: string;
    extraQueryParams?: string;
    scope?: string;
    audience?: string;
    idleSessionLifetimeInSeconds?: number;
};
```

If no `oidcConfiguration` is provided for a service, Onyxia will **reuse its own OIDC client**\
and the same Access Token for authentication. However, it is **recommended** to provide\
a **separate client ID** for each service to improve access control and security.

Example configuration in `values.yaml`:

{% code title="" %}
```yaml
onyxia:
  api:
    env:
      authentication.mode: "openidconnect"
      oidc.issuer-uri: "https://auth.lab.my-domain.net/auth/realms/datalab"
      oidc.clientID: "onyxia"
    regions: 
      [
        {
          data: {
            S3: {
              sts: {
                oidcConfiguration: {
                  clientID: "onyxia-minio",
                }
              }
            }
          },
          vault: {
            oidcConfiguration: {
              clientID: "onyxia-vault"
            }
          },
          services: {
            k8sPublicEndpoint: {
              oidcConfiguration: {
                clientID: "onyxia-k8s"
              }
            }
          }
        }
      ]
```
{% endcode %}

⚠ ️ Important: Consistency of Claims Across Services:

When configuring OIDC for Onyxia, you define specific claims that indicate where to find\
the **user identifier**, **groups**, and **roles** within the Access Token's JWT.

These claims **cannot be configured separately for each service** Onyxia interacts with (e.g., S3, Vault, Kubernetes API).\
They must remain **consistent across all OIDC-enabled services** to ensure proper authentication and authorization.

When a user logs in, the OIDC provider issues an Access Token for the `onyxia` client.\
This token includes claims such as:

```json
{
  "sub": "abcd1234",
  "preferred_username": "jhondoe",
  "groups": [ "funathon", "spark-lab" ],
  "roles": [ "vip", "admin-keycloak" ],
}
```

If you have configured `oidc.username-claim: "preferred_username"` in the main Onyxia configuration,\
Onyxia expects that all other services it interacts with—such as `onyxia-minio`, `onyxia-vault`, and `onyxia-k8s`—\
will also receive Access Tokens where the **same claim (`preferred_username`) exists and holds the same value**.

To avoid any issue, **all OIDC clients** (`onyxia`, `onyxia-minio`, `onyxia-vault`, `onyxia-k8s`)\
should be configured within **the same SSO realm** in your OIDC provider.\
This ensures that each issued Access Token follows the same claim structure and contains\
consistent values for the same user.

If you're unsure whether your setup meets this requirement, **check the JWT of each Access Token**\
issued for different clients and confirm that the claims are aligned.
