---
description: Enable AI providers, connect an OpenWebUI gateway, and expose AI credentials to service charts.
icon: robot
---

# AI integration

Onyxia can centralize AI provider settings for a user and inject the selected provider into compatible services at launch time.

The feature supports two kinds of providers:

* **Region providers** are OpenWebUI gateways configured by the platform administrator. Onyxia exchanges the user's OIDC access token for a short-lived OpenWebUI token.
* **Custom providers** are configured by users. OpenAI, OpenAI-compatible, Mistral, and Anthropic API protocols are supported.

{% hint style="info" %}
Enabling the feature adds **My account > AI**. It does not add AI support to every service automatically. A Helm chart must use the [`ai` x-onyxia context](#inject-the-provider-into-a-service) to receive the selected provider.
{% endhint %}

## Enable the feature

The feature is disabled by default. Enable it in the Onyxia Web configuration:

{% code title="apps/onyxia/values.yaml" %}
```yaml
onyxia:
  web:
    env:
      ENABLED_AI: "true"
```
{% endcode %}

This is the master switch for both region providers and custom providers. You can enable it without configuring a region provider; users will then only be able to add custom providers.

The AI tab requires an authenticated user. User preferences and custom provider credentials are stored with the other Onyxia user settings:

* in the user's Vault-backed configuration when the region has Vault;
* in browser local storage when the region has no Vault.

## Configure an OpenWebUI gateway

Add `data.ai` to the existing region configuration as a JSON-style array. In `values.yaml`, the flow syntax used below is valid YAML.

{% code title="apps/onyxia/values.yaml (region excerpt)" %}
```yaml
onyxia:
  api:
    regions: [
      {
        # Keep the other required region properties and existing data configuration here.
        data: {
          ai: [
            {
              id: "openwebui",
              name: "Organization AI gateway",
              URL: "https://ai.example.com",
              oauthProvider: "oidc",
              description: {
                en: "Use the models hosted by our organization.",
                fr: "Utilisez les modèles hébergés par notre organisation."
              },
              accountCreation: {
                title: {
                  en: "Activate your AI account",
                  fr: "Activez votre compte IA"
                },
                description: {
                  en: "Open the gateway and sign in once, then return to Onyxia.",
                  fr: "Ouvrez la passerelle et connectez-vous une première fois, puis revenez dans Onyxia."
                },
                buttonLabel: {
                  en: "Open the gateway",
                  fr: "Ouvrir la passerelle"
                },
                logoURL: "https://ai.example.com/static/logo.png"
              },
              oidcConfiguration: {
                clientID: "onyxia-ai",
                issuerURI: "https://auth.example.com/realms/example"
              }
            }
          ]
        }
      }
    ]
```
{% endcode %}

Do not add a trailing slash to `URL`. Onyxia derives the API base URL as `<URL>/api`.

### Gateway properties

| Property | Required | Description |
| --- | --- | --- |
| `URL` | Yes | Public base URL of the OpenWebUI instance. The user's browser must be able to reach it. |
| `oauthProvider` | Yes | OpenWebUI OAuth provider identifier used in `/api/v1/auths/oauth/<provider>/token/exchange`; commonly `oidc`. |
| `id` | Recommended | Stable, unique identifier used to persist the user's model and default-provider selections. If omitted, it is derived from the gateway's position in the list. |
| `name` | No | Label displayed in Onyxia. Defaults to the hostname from `URL`. |
| `provider` | No | Protocol name injected into charts. Defaults to `openai`, which is appropriate for the OpenWebUI OpenAI-compatible API. |
| `description` | No | String or localized Markdown displayed below the gateway name. |
| `accountCreation` | No | Localized title, description, and button label, plus an optional `logoURL`, displayed when OpenWebUI returns `403` because the user has no account yet. The button opens `URL`. |
| `oidcConfiguration` | No | OIDC overrides for this gateway: `issuerURI`, `clientID`, `extraQueryParams`, `scope`, or `idleSessionLifetimeInSeconds`. Unspecified values are inherited from the main Onyxia OIDC configuration. |

Use an explicit, stable `id` for every gateway. Changing it makes Onyxia treat the gateway as a new provider and discards the model selection associated with the previous identifier.

### Configure OpenWebUI

Onyxia uses the following OpenWebUI endpoints directly from the user's browser:

* `POST <URL>/api/v1/auths/oauth/<oauthProvider>/token/exchange` with `{ "token": "<OIDC access token>" }`;
* `GET <URL>/api/models` with the returned token as a Bearer credential.

Configure OpenWebUI to enable token exchange, trust the OIDC client used by Onyxia, and allow the Onyxia origin through CORS:

```dotenv
ENABLE_OAUTH_TOKEN_EXCHANGE=true
OAUTH_TOKEN_EXCHANGE_TRUSTED_CLIENT_IDS=onyxia-ai
CORS_ALLOW_ORIGIN=https://onyxia.example.com
```

Create `onyxia-ai` as a public OIDC client using Authorization Code Flow with PKCE. Configure the Onyxia URL as an allowed redirect URI. A dedicated client is recommended; if `oidcConfiguration` is omitted, add Onyxia's main client ID to `OAUTH_TOKEN_EXCHANGE_TRUSTED_CLIENT_IDS` instead.

{% hint style="warning" %}
Configure the OIDC client used for the AI gateway so that the identity provider issues standard Bearer access tokens; do not require DPoP-bound access tokens for this client. Onyxia hands the access token to OpenWebUI's token-exchange endpoint, which reuses it without access to the private key held by the browser and therefore cannot present the associated DPoP proof. This restriction only applies to the AI client; other Onyxia OIDC clients can still use DPoP.
{% endhint %}

The first exchange can return `403` if the user does not yet exist in OpenWebUI. In that case, Onyxia displays the account-creation content. The user must open the gateway, sign in once, return to Onyxia, and select **Refresh credentials**.

## Inject the provider into a service

The launcher exposes the user's AI configuration through the [`x-onyxia`](catalog-of-services/custom-catalogs/onyxia-extension.md) context:

| Context path | Value |
| --- | --- |
| `ai.enabled` | `true` when at least one usable provider is available. |
| `ai.activeProvider` | The provider selected as default, or `undefined`. |
| `ai.providers` | Other usable providers; the active provider is not repeated in this list. |

Each provider contains `id`, `isDefault`, `name`, `provider`, `apiBase`, `apiKey`, `selectedModel`, and, when model discovery succeeded, `models`.

The chart decides how these values map to its own `values.yaml`. The following JSON Schema fragment injects the default provider into an `ai` values object:

{% code title="values.schema.json" %}
```json
{
  "ai": {
    "type": "object",
    "properties": {
      "enabled": {
        "type": "boolean",
        "default": false,
        "x-onyxia": {
          "overwriteDefaultWith": "{{ai.enabled}}",
          "hidden": true
        }
      },
      "provider": {
        "type": "string",
        "default": "",
        "x-onyxia": {
          "overwriteDefaultWith": "{{ai.activeProvider.provider}}",
          "hidden": true
        }
      },
      "apiBase": {
        "type": "string",
        "default": "",
        "x-onyxia": {
          "overwriteDefaultWith": "{{ai.activeProvider.apiBase}}",
          "hidden": true
        }
      },
      "apiKey": {
        "type": "string",
        "default": "",
        "render": "password",
        "x-onyxia": {
          "overwriteDefaultWith": "{{ai.activeProvider.apiKey}}",
          "hidden": true
        }
      },
      "model": {
        "type": "string",
        "default": "",
        "x-onyxia": {
          "overwriteDefaultWith": "{{ai.activeProvider.selectedModel}}",
          "hidden": true
        }
      }
    }
  }
}
```
{% endcode %}

Define matching defaults in `values.yaml` and only create AI-related environment variables or Secrets when `ai.enabled` is `true`.

{% hint style="warning" %}
`apiKey` is sensitive. Once injected, it becomes part of the Helm values used to launch the service. Store it in a Kubernetes Secret, never a ConfigMap, and do not print it in templates, logs, notes, or post-install instructions.
{% endhint %}

## Validation checklist

1. Sign in to Onyxia and open **My account > AI**.
2. Confirm that the gateway appears with the expected name and description.
3. If prompted, open OpenWebUI and sign in once, then refresh the credentials in Onyxia.
4. Confirm that `GET <URL>/api/models` loads the model selector.
5. Select a default provider and model.
6. Launch a compatible chart and inspect its generated Helm values to confirm the expected mapping.

If the AI tab is missing, verify `ENABLED_AI`. If token exchange or model loading fails, check the browser network panel, OpenWebUI's trusted client list, the `oauthProvider` identifier, and CORS for the exact Onyxia origin.
