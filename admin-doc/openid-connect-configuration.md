---
hidden: true
icon: key-skeleton
---

# OpenID Connect Configuration

The [installation guide](readme/user-authentication.md) configures Keycloak with one public `onyxia` client and one access token accepted by the services Onyxia uses. This page documents the available OIDC settings, including overrides for services that need a separate token.

Onyxia Web is a SPA and uses a **public client with Authorization Code and PKCE**. It has no client secret.

## API Reference

<details>

<summary>Overview of all the available parameters</summary>

{% code title="apps/onyxia/values.yaml" %}
```yaml
onyxia:
  api:
    env:
      # Enable OIDC authentication.
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
      # The Keycloak reference realm supplies `onyxia-username` through the
      # RFC1123 username mapper. Other claims are suitable if their values are
      # unique, stable, and compatible with resource naming.
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
      # Use scope or extra query parameters to request a token for that audience.
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

## Provider configuration

### Keycloak

Use the [reference realm](https://github.com/garronej/onyxia-ops/blob/main/apps/keycloak/realm-export.json) and [Onyxia values](https://github.com/garronej/onyxia-ops/blob/main/apps/onyxia/values-keycloak-enabled.yaml) from the installation guide. The client ID is `onyxia`, the API audience is `onyxia-api`, and the username claim is `onyxia-username`.

The [RFC1123 username mapper](https://github.com/onyxia-datalab/keycloak-rfc1123-username) provides a stable, unique identifier without restricting the user's login name. A registration regexp is an alternative for a new user base, but does not repair existing usernames.

For general public-client setup and a test application, see the [oidc-spa Keycloak guide](https://docs.oidc-spa.dev/providers-configuration/keycloak).

### Other IAM systems

[Microsoft Entra ID](https://learn.microsoft.com/en-us/entra/msal/javascript/browser/resources-and-scopes) and [Auth0](https://auth0.com/docs/secure/tokens/access-tokens/get-access-tokens) require separate tokens for distinct resources. Onyxia can request those tokens through scope or audience configuration, including with the same public client ID.

For provider-specific application setup, see the [Entra ID](https://docs.oidc-spa.dev/providers-configuration/microsoft-entra-id) and [Auth0](https://docs.oidc-spa.dev/providers-configuration/auth0) guides. **Contact the Onyxia team for help integrating any IAM system other than Keycloak**, including the username claim and service authorization settings.

## OIDC Configuration for Services Onyxia Connects To

By default, services reuse the main `onyxia` access token. This is the model used by the Keycloak installation guide; leave service-specific `oidcConfiguration` objects unset when using its reference realm.

When a service needs a different issuer, client, scope, or requested audience, supply an `oidcConfiguration` override in the region:

- **S3 (STS):** `onyxia.api.regions[].data.S3.sts.oidcConfiguration`
- **Vault:** `onyxia.api.regions[].vault.oidcConfiguration`
- **Kubernetes API:** `onyxia.api.regions[].services.k8sPublicEndpoint.oidcConfiguration`

Fields are optional; omitted values inherit the main OIDC configuration:

```ts
type OidcConfiguration = {
    issuerURI?: string;
    clientID?: string;
    scope?: string;
    extraQueryParams?: string;
    idleSessionLifetimeInSeconds?: number | string;
};
```

For example, a service can request an Entra ID resource scope with `scope: "openid profile api://vault/access_as_user"`, or an Auth0 audience with `extraQueryParams: "audience=https%3A%2F%2Fvault.lab.my-domain.net"`. These change the token request. The API's `oidc.audience` setting only validates incoming tokens; it does not request a different token from the identity provider.
