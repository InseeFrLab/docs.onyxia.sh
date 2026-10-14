---
icon: key-skeleton
---

# OpenID Connect Configuration

[The installation guide](readme/user-authentication.md) explain how to set up a new [Keycloak](https://www.keycloak.org/) instance to enable authentication on your datalab.

However, chances are that your organization already has an existing IAM system in place. This guide covers how to integrate Onyxia with various commonly used OIDC providers, including [Keycloak](https://www.keycloak.org/), [Auth0](https://auth0.com/), and [Microsoft Entra ID](https://www.microsoft.com/en-us/security/business/identity-access/microsoft-entra-id).

{% hint style="warning" %}
Onyxia use **Public** OpenID Connect client: **no client Secret**.

The technical term for a public OIDC client is **Authorization Code Flow + PKCE**.

It's the type of client that you create for Single Page Application (SPA).
{% endhint %}

## API Reference

<details>

<summary>Overview of all the available parameters</summary>

{% code title="apps/onyxia/values.yaml" %}
```yaml
onyxia:
  api:
    env:
      # Mandatory and no other authentication mode is currently supported.
      authentication.mode: "openidconnect"

      # Mandatory: The issuer URI of the OIDC provider.  
      oidc.issuer-uri: "..."

      # Mandatory: The client ID of the OIDC client representing the Onyxia Web Application.
      oidc.clientID: "..."

      # Mandatory: Defines which claim in the Access Token's JWT serves as the unique 
      # user identifier.  
      # This identifier must contain only lowercase alphanumeric characters and `-`. 
      # Specifically, it must comply with RFC 1123: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#dns-label-names
      #
      # - If your usernames already conform to this constraint, you can use 
      #   `"preferred_username"` for a more human-readable identifier.
      # - If usernames contain special characters, use another claim 
      #   such as `"sub"` (Ensure that the `sub` values comply with RFC 1123).  
      #
      oidc.username-claim: "..."

      # Optional: Defaults to `"groups"`. Defines which claim represents user groups.
      # See: https://docs.onyxia.sh/admin-doc/setting-up-group-projects
      oidc.groups-claim: "..."

      # Optional: Defaults to `"roles"`. Defines which claim represents user roles.
      oidc.roles-claim: "..."

      # Optional: Additional query parameters to append to the OIDC authorization
      # endpoint (the login url).   
      # Example: If using Keycloak with Google OAuth as an identity provider, you might want  
      # to preselect Google as the login option using `"kc_idp_hint=google"`.  
      # 
      # ⚠️ This string is appended as-is. Ensure it is properly URI-encoded.  
      # If adding multiple parameters, separate them with `&`.  
      #
      # Example: `"foo=foo%20value&bar=bar%20value"`
      #
      # duct-taping case: If you provide an audience as query param like
      # `"audience=onyxia"`, the audience will also be passed as an extra
      # token param because some AS might expect it.  
      oidc.extra-query-params: "..."

      # Optional: Expected audience (`aud`) value in the Access Token.  
      # If set, Onyxia-API validates the `aud` claim and rejects requests
      # where it doesn’t match (or isn’t included if `aud` is an array).  
      # This setting applies only on the server side.
      # Defining it here won’t change how the OIDC client requests tokens.  
      # Refer to your provider’s documentation below for details.
      oidc.audience: "..."

      # Optional: Specifies the OIDC scopes requested by the Onyxia client.  
      # Defaults to `"openid profile"`.  
      # This is a space-separated list. `"openid"` is always requested, 
      # regardless of this setting.
      oidc.scope: "..."
      
      # Optional: Automatically logs out users after a set period of inactivity. 
      # If you are using Keycloak do not provide this value, it's inferred automatically. 
      oidc.idleSessionLifetimeInSeconds: "..."

      # Optional: The Onyxia API fetches `<issuer-uri>/.well-known/openid-configuration` 
      # to retrieve JWKs for validating Access Tokens (used as Authorization Bearers).  
      #
      # ⚠️ In development, if you lack proper root certificates, you can disable TLS verification.  
      # However, in production, it is strongly recommended to mount the correct `cacerts` instead.
      oidc.skip-tls-verify: "true|false"
```
{% endcode %}

</details>

***

## OIDC Provider Specific Configuration Guides

{% tabs %}
{% tab title="Keycloak" %}
**Choosing the Unique User Identifier Claim**

Onyxia requires a unique user identifier. You must specify which claim in the Access Token should be used for this purpose.

Ideally, you can use `preferred_username` as an identifier, but this requires ensuring it complies with [RFC 1123](https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#dns-label-names). This means it must contain only lowercase alphanumeric characters and `-`.

Since this format is restrictive, if you already have an existing user base, `preferred_username` may not be an option. In that case, you have two alternatives:

* **Define a custom claim**: Configure a Keycloak mapper to generate an RFC 1123-compliant claim in the Access Token.
* **Use `"sub"`**: This claim is guaranteed to be unique and always present, but ensure that the `sub` values comply with RFC 1123.

If you are starting fresh with no existing users, you can enforce a regex pattern in the **User Profile Attributes** to require usernames that comply with the restriction.

More details can be found in [the installation guide](https://docs.onyxia.sh/admin-doc/readme/user-authentication) (search for "pattern").

**Configuring Keycloak**

Beyond what's covered in the installation guide, if you need a more general tutorial on setting up a public Keycloak OIDC client like Onyxia, refer to the following guide. It includes a test project to validate your configuration.

{% embed url="https://docs.oidc-spa.dev/providers-configuration/keycloak" %}
For Onyxia, use these substitutions in the guide:\
**\<KC\_DOMAIN>**: `auth.lab.my-domain.net`\
**\<KC\_RELATIVE\_PATH>**: `/auth`\
**\<REALM\_NAME>**: `datalab`\
**\<APP\_DOMAIN>**: `datalab.my-domain.net`\
**\<BASE\_URL>**: `/`\
**\<DEV\_PORT>**: `5173`\
✅ Note that Onyxia implement an auto logout countdown that will start to display once minute befor auto logout if you configure your client as [a sensible app](https://docs.oidc-spa.dev/providers-configuration/keycloak#security-sensitive-apps-banking-admin-panels-etc)
{% endembed %}

Here is an overview of what your Onyxia `values.yaml` should look like:

{% code title="apps/onyxia/values.yaml" %}
```yaml
onyxia:
  api:
    env:
      authentication.mode: "openidconnect"
      oidc.issuer-uri: "https://auth.lab.my-domain.net/auth/realms/datalab"
      # Warning: This is the client ID used by onyxia-web (the fronted)
      # Onyxia-api does not need any client, just to validate an audience.
      oidc.clientID: "onyxia_onyxia-api"
      oidc.audience: "onyxia-api"
      # The username should be RFC1123 complient
      oidc.username-claim: "preferred_username"
    regions: 
      [
        {
          data: {
            S3: {
              sts: {
                oidcConfiguration: { clientID: "onyxia_minio" }
              }
            }
          },
          vault: {
            oidcConfiguration: { clientID: "onyxia_vault" }
          },
          services: {
            k8sPublicEndpoint: {
              oidcConfiguration: { clientID: "onyxia_kub-api-server" }
            }
          }
        }
      ]
```
{% endcode %}
{% endtab %}

{% tab title="Microsoft Entra ID" %}
Follow this guide to configure a Microsoft Entra ID application for Onyxia.

Application to declare:

* onyxia-api, exposed api: Application ID URI: api://onyxia-api scope: access\_as\_suer
* onyxia\_onyxia-api, Platform: Single-Page Application, Redirect URIs: https://datalab.my-domain.net/ https://datalab.my-domain.net/api/swagger-ui/oauth2-redirect.html https://datalab.my-domain.net/api/swagger-ui/oauth2-redirect.html, with api permission on onyxia-api
* minio, exposed api: Application ID URI: api://minio scope: access\_as\_user
* onyxia\_onyxia-api, Platform: Single-Page Application, Redirect URIs: https://datalab.my-domain.net/, with api permission on minio
* TODO: AI please infer<br>

More info on how to navi

{% embed url="https://docs.oidc-spa.dev/providers-configuration/microsoft-entra-id" %}
For Onyxia, use these substitutions:\
`My App - API` -> `onyxia-api`\
`api://my-app-api` -> `api://onyxia-api`\
`My App` -> `onyxia_onyxia-api`\
[`https://my-app.com/`](https://my-app.com/) -> `https://datalab.my-domain.net/`
{% endembed %}

Here is what your configuration should look like:

{% code title="apps/onyxia/values.yaml" %}
```yaml
onyxia:
  api:
    env:
      authentication.mode: "openidconnect"
      oidc.issuer-uri: "https://login.microsoftonline.com/<Directory (tenant) ID>/v2.0"
      # Warning: This is the client ID used by onyxia-web (the fronted)
      # Onyxia-api does not need any client, just to validate an audience.
      oidc.clientID: "<Application ID: onyxia_onyxia-api>"
      # Do **not** use `"sub"` or `"upn"` as they may contain 
      # non-alphanumeric characters.
      oidc.username-claim: "oid"
      oidc.scope: "profile api://onyxia-api/access_as_user"
      oidc.audience: "<Application ID: onyxia-api>"
    regions: 
      [
        {
          data: {
            S3: {
              sts: {
                oidcConfiguration: { 
                  clientID: "<Application ID: onyxia_minio>",
                  scope: "profile api://onyxia/access_as_user" 
                }
              }
            }
          },
          vault: {
            oidcConfiguration: { 
              clientID: "<Application ID: onyxia_vault",
              scope: "profile api://vault/access_as_user"
            }
          },
          services: {
            k8sPublicEndpoint: {
              oidcConfiguration: { 
                clientID: "<Application ID: onyxia_kub-api-server>" ,
                scope: "profile api://kub-api-server/access_as_user"
              }
            }
          }
        }
      ]
      
```
{% endcode %}
{% endtab %}

{% tab title="Auth0" %}
Follow this guide to configure an Auth0 application for Onyxia.

{% embed url="https://docs.oidc-spa.dev/providers-configuration/auth0" %}
For Onyxia, use these substitutions:\
`"My App"` → `"Onyxia"`\
**\<APP\_DOMAIN>** → `datalab.my-domain.net`\
\&#xNAN;**\<BASE\_URL>** → `/`\
\&#xNAN;**\<DEV\_PORT>** → `5173`\
`"My App - API"` → `"Onyxia - API"`\
`https://myapp.my-company.com/api` → `https://datalab.my-domain.net/api`\
`"auth.my-company.com"` → `"auth.my-domain.net"`
{% endembed %}

**Generating an RFC 1123-Compliant Claim in the Access Token**

By default, Auth0 does not issue a claim that Onyxia can use as a unique user identifier. You must create one by defining a **custom claim** in the access token using an Auth0 **Trigger Action**.

**Steps to Create the `onyxia-username` Claim**

1️⃣ **Create a Custom Action**:

1. Go to **Auth0 Dashboard** → **Actions** → **Library**.
2. Click **Create Action**.
3. Set:
   * **Name**: `GenerateOnyxiaUsername`
   * **Trigger**: **Post Login**
   * **Runtime**: `Node 22`
4. Click **Create**.

2️⃣ **Add the Custom Code**:\
Replace the default content with:

```js
function toRFC1123(input) {
  if (!input) return "";
  let output = input.toLowerCase().replace(/[^a-z0-9-]/g, "-").replace(/^-+|-+$/g, "");
  return output.length > 63 ? output.substring(0, 63).replace(/-+$/, "") : output;
}

exports.onExecutePostLogin = async (event, api) => {
  const sub = event.user.user_id;
  if (sub) api.accessToken.setCustomClaim("onyxia-username", toRFC1123(sub));
};
```

3️⃣ **Deploy and Activate the Action**:

1. Click **Deploy**.
2. Go to **Auth0 Dashboard** → **Actions** → **Triggers** → **Post Login**.
3. Drag & drop `GenerateOnyxiaUsername` into the flow.
4. Click **Apply Changes**.

Now, your access token will include the `onyxia-username` claim.

<figure><img src="../.gitbook/assets/image (2).png" alt="" width="375"><figcaption><p>Preview of the decoded JWT of the Access Token issued by Auth0<br>with the custom action enabled when previewed with the<br>test app of the oidc-spa guide</p></figcaption></figure>

**Final Configuration**

{% code title="apps/onyxia/values.yaml" %}
```yaml
onyxia:
  api:
    env:
      authentication.mode: "openidconnect"
      oidc.issuer-uri: "https://auth.my-domain.net"
  
      # onyxia-api specific params (not used by the frontend)
      oidc.audience: "https://datalab.my-domain.net/api"
      oidc.username-claim: "onyxia-username"
      
      # Client sepecifc parameters (not used by the backend)
      oidc.clientID: "<Application Client ID: onyxia_onyxia-api>"  
      oidc.extra-query-params: "audience=https%3A%2F%2Fdatalab.my-domain.net%2Fapi"
      # Optional: Auto logout after inactivity.
      oidc.idleSessionLifetimeInSeconds: "300"
    regions: 
      [
        {
          data: {
            S3: {
              sts: {
                oidcConfiguration: { 
                  clientID: "<Application Client ID: onyxia_minio>",
                  extraQueryParams: "audience=https%3A%2F%2Fminio.lab.my-domain.net" 
                }
              }
            }
          },
          vault: {
            oidcConfiguration: { 
              clientID: "<Application Client ID: onyxia_vault",
              extraQueryParams: "audience=https%3A%2F%2Fvault.lab.my-domain.net" 
            }
          },
          services: {
            k8sPublicEndpoint: {
              oidcConfiguration: { 
                clientID: "<Applicatioin Client ID: onyxia_kub-api-server>",
                extraQueryParams: "audience=https%3A%2F%2Fapiserver.kub.sspcloud.fr" 
              }
            }
          }
        }
      ]
```
{% endcode %}
{% endtab %}

{% tab title="Other" %}
Please [contact US](https://join.slack.com/t/3innovation/shared_invite/zt-2skhjkavr-xO~uTRLgoNOCm6ubLpKG7Q) or infer from Auth0 documentations.
{% endtab %}
{% endtabs %}

## **OIDC Configuration for Services Onyxia Connects To**

Onyxia uses an OIDC client for authentication, but it also connects to other OIDC-enabled services.\
Each of these services **can** have its own OIDC client instance configuration, allowing Onyxia to authenticate using a separate client identity.

In the **region configuration**, you can specify an optional `oidcConfiguration` object for\
each service:

* **S3 (MinIO STS)** → `onyxia.api.regions[].data.S3.sts.oidcConfiguration`
* **Vault** → `onyxia.api.regions[].vault.oidcConfiguration`
* **Kubernetes API** → `onyxia.api.regions[].services.k8sPublicEndpoint.oidcConfiguration`

Each configuration follows this structure:

```ts
type OidcConfiguration = {
    clientID: string;
    extraQueryParams?: string;
    scope?: string;
};
```

If no `oidcConfiguration` is provided for a service, Onyxia will reuse the same access\_token used for onyxia-api.

However, defining a separate OIDC client for each service is recommended to improve access control and security.

You might find it strange that Onyxia requires creating multiple OIDC clients to communicate with different resource servers (e.g. `onyxia-api`, `minio`, `vault`, or the Kubernetes API). You’ll typically end up with several clients such as `onyxia`, `onyxia-vault`, `onyxia-minio`, and `onyxia-kube`.\
At first, this can feel counterintuitive, a _client ID_ seems like it should represent one application, not multiple variants of it.

Conceptually, a single client requesting tokens for multiple resource servers (each with its own audience and claims) would make more sense.\
However, Keycloak doesn’t model things that way. While Onyxia supports any OpenID Connect provider, it’s primarily designed around Keycloak’s behavior and limitations.

In Keycloak’s traditional model, an OIDC _client_ actually represents **an application talking to a specific resource server**, not just an application itself.



***
