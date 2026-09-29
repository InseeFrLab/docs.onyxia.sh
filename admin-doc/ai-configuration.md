---
description: Configure AI providers, authentication, model selection, and AI settings for service charts.
icon: robot
---

# AI integration

Onyxia centralizes AI provider settings for each user and passes their selected models and providers to compatible services at launch time.

Administrators can configure providers with no authentication, a user-supplied API key, or an OpenWebUI OIDC token exchange. Users can also add their own providers unless the administrator disables this option. Supported provider types are OpenAI-compatible, OpenAI, Anthropic, Mistral, and DeepSeek.

{% hint style="info" %}
The feature adds **My account > AI**. A Helm chart must use the [`ai` x-onyxia context](#inject-the-provider-into-a-service) to configure AI in a service. It does not add AI capabilities to every service automatically.
{% endhint %}

## Enable or disable AI

AI is enabled by default for authenticated users. Configure it through the `AI` environment variable in Onyxia Web. The value is a JSON5 **object**, containing a `providers` property. In Helm values, use a YAML literal block (`|`) to pass it as a string.

An unset or empty `AI` value enables the feature with no administrator-configured providers; users can add their own. To disable the AI tab and its launch context:

{% code title="apps/onyxia/values.yaml" %}
```yaml
onyxia:
  web:
    env:
      AI: |
        { disable: true, providers: [] }
```
{% endcode %}

| Property | Default | Description |
| --- | --- | --- |
| `disable` | `false` | Disable AI integration. |
| `disallowUserToAddProviders` | `false` | Hide the option to add custom providers. This does not remove previously saved custom providers. |
| `description` | Unset | Introductory Markdown below the AI Providers heading; a string or a localized object such as `{ en: "...", fr: "..." }`. |
| `providers` | Required for a nonempty configuration | An array of provider objects, or a single provider object. Use `[]` for none. |

{% hint style="warning" %}
The `AI` configuration is exposed to the browser. Do not put API keys, client secrets, or other secrets in it. User-supplied API keys are entered through **My account > AI**.
{% endhint %}

## Configure providers

This example offers an OpenWebUI gateway and a provider for which each user supplies their own API key:

{% code title="apps/onyxia/values.yaml" %}
```yaml
onyxia:
  web:
    env:
      AI: |
        {
          description: {
            en: "Choose the models available in your services.",
            fr: "Choisissez les modèles disponibles dans vos services."
          },
          providers: [
            {
              name: "Organization AI",
              providerType: "openai-compatible",
              apiBase: "https://ai.example.com/api",
              authentification: {
                type: "api-key",
                obtentionMethod: "open-webui-oidc-token-exchange",
                oidcConfiguration: {
                  clientID: "onyxia-ai",
                  issuerURI: "https://auth.example.com/realms/example"
                }
              },
              documentation: {
                mainText: "Sign in to the gateway once before using it from Onyxia.",
                links: [
                  { label: "Open the gateway", url: "https://ai.example.com" }
                ]
              }
            },
            {
              name: "Personal OpenAI key",
              providerType: "openai",
              apiBase: "https://api.openai.com/v1",
              authentification: {
                type: "api-key",
                obtentionMethod: "user-provided"
              }
            }
          ]
        }
```
{% endcode %}

### Provider properties

| Property | Required | Description |
| --- | --- | --- |
| `name` | Yes | Unique provider name, without `/`. Also serves as its identifier in saved preferences and the launch context. Keep it stable. |
| `providerType` | Yes | `openai-compatible`, `openai`, `anthropic`, `mistral`, or `deepseek`. |
| `apiBase` | Yes | Full API base URL, including `/api` or `/v1` where appropriate. Onyxia removes trailing slashes. |
| `authentification` | Yes | One of the authentication objects below. Use this exact spelling. |
| `models` | No | Explicit array of model IDs. When omitted, models come from the provider's `/models` endpoint. |
| `documentation` | No | Help displayed when managing the provider: `mainText` (Markdown) and optional `links`, each with `label` and `url`. Text and labels accept strings or localized objects. |
| `logoUrl` | No | Image URL, or `{ light: "https://...", dark: "https://..." }`. |

Renaming an administrator-configured provider changes its identity: keys and model preferences saved under its previous name are not transferred. If its name conflicts with an existing custom provider, the custom provider remains visible but is excluded from the launch context until renamed or deleted.

### Authentication methods

| `authentification` value | Behavior |
| --- | --- |
| `{ type: "none" }` | No credential is required or injected. |
| `{ type: "api-key", obtentionMethod: "user-provided" }` | Each user supplies their own key in the provider's **Manage** panel. |
| `{ type: "api-key", obtentionMethod: "open-webui-oidc-token-exchange" }` | Onyxia exchanges an OIDC access token for an OpenWebUI token. Optional OIDC overrides belong inside this object. |

### Model discovery and selection

Onyxia requests `GET <apiBase>/models` from the user's browser. Providers must therefore be reachable from the browser and permit the Onyxia origin through CORS. Requests use Bearer authentication for OpenAI-compatible protocols, or Anthropic's native headers for `anthropic`.

An explicit `models` list supplies the available model IDs even when discovery fails. Onyxia still attempts discovery to display the connection status, so a card can show a connection error while its configured models remain selectable. Services must be able to reach the provider, and authentication must still be available.

Users can select several models per provider. All models are initially selected; newly discovered models are selected unless previously excluded. A provider with no selected model is not injected into services. The global default is a **model**, identified by both provider name and model ID.

## Configure OpenWebUI

For `open-webui-oidc-token-exchange`, set `apiBase` to the OpenWebUI API URL, for example `https://ai.example.com/api`. Onyxia calls:

- `POST <apiBase>/v1/auths/oauth/oidc/token/exchange` with `{ "token": "<OIDC access token>" }`;
- `GET <apiBase>/models` with the returned token as a Bearer credential.

The OAuth provider identifier in this exchange path is fixed to `oidc`.

Enable token exchange in OpenWebUI and restrict trusted clients to the OIDC client used by Onyxia. Allow the Onyxia origin through CORS:

```dotenv
ENABLE_OAUTH_TOKEN_EXCHANGE=true
OAUTH_TOKEN_EXCHANGE_TRUSTED_CLIENT_IDS=onyxia-ai
CORS_ALLOW_ORIGIN=https://onyxia.example.com
```

Check the prerequisites for your OpenWebUI version in its [SSO documentation](https://docs.openwebui.com/features/authentication-access/auth/sso/), including token introspection for trusted-client validation.

Create `onyxia-ai` as a public OIDC client using Authorization Code Flow with PKCE, with the Onyxia URL as an allowed redirect URI. The optional `authentification.oidcConfiguration` supports `issuerURI`, `clientID`, `extraQueryParams`, `scope`, and `idleSessionLifetimeInSeconds`. Unspecified values inherit the main Onyxia OIDC configuration. If no dedicated client is configured, trust Onyxia's main client ID in OpenWebUI instead.

Onyxia disables DPoP for this token-exchange flow. Configure the associated OIDC client to allow standard Bearer access tokens. Other Onyxia OIDC clients can still use DPoP.

A user may need to sign in to OpenWebUI once before exchanging tokens. Use `documentation.mainText` and `documentation.links` to explain this prerequisite and link to your gateway. Exchange failures appear as connection errors; there is no separate account-creation screen.

Exchanged tokens are kept in runtime state, not persisted as user-supplied keys. Onyxia obtains fresh tokens when preparing the launch context. It does not update credentials already injected into running services.

## Inject the provider into a service

The launcher exposes the following [`x-onyxia`](catalog-of-services/custom-catalogs/onyxia-extension.md) context:

| Context path | Value |
| --- | --- |
| `ai.enabled` | `true` when at least one model from a usable provider can be injected. |
| `ai.models` | Selected models from usable providers, as `<providerName>/<modelId>` strings. |
| `ai.defaultModel` | Default model in the same format, or `undefined`. Falls back to the first injectable model if the chosen default is unavailable. |
| `ai.providers` | All usable providers, including the provider of the default model. |

Each provider contains `name`, `apiBase`, `apiKey` (possibly `undefined`), `models` (selected model IDs without the provider prefix), and `type` (the configured `providerType`). Providers requiring authentication are included only when a credential is available. Name conflicts and empty model selections exclude a provider.

This JSON Schema fragment belongs under the root `properties` of a chart's `values.schema.json`:

{% code title="values.schema.json (fragment)" %}
```json
{
  "ai": {
    "type": "object",
    "properties": {
      "enabled": {
        "type": "boolean",
        "default": false,
        "x-onyxia": { "overwriteDefaultWith": "{{ai.enabled}}" }
      },
      "selectedModel": {
        "type": "string",
        "default": "",
        "listEnum": [],
        "x-onyxia": {
          "overwriteDefaultWith": "{{ai.defaultModel}}",
          "overwriteListEnumWith": "{{ai.models}}"
        }
      },
      "providers": {
        "type": "array",
        "default": [],
        "items": {
          "type": "object",
          "properties": {
            "name": {
              "type": "string",
              "default": "",
              "x-onyxia": { "overwriteDefaultWith": "{{name}}" }
            },
            "apiBase": {
              "type": "string",
              "default": "",
              "x-onyxia": { "overwriteDefaultWith": "{{apiBase}}" }
            },
            "apiKey": {
              "type": "string",
              "default": "",
              "render": "password",
              "x-onyxia": { "overwriteDefaultWith": "{{apiKey}}" }
            },
            "models": {
              "type": "array",
              "default": [],
              "items": { "type": "string" },
              "x-onyxia": { "overwriteDefaultWith": "{{models}}" }
            },
            "type": {
              "type": "string",
              "default": "",
              "x-onyxia": { "overwriteDefaultWith": "{{type}}" }
            }
          }
        },
        "x-onyxia": {
          "overwriteDefaultWith": "{{ai.providers}}",
          "hidden": true
        }
      }
    }
  }
}
```
{% endcode %}

Define matching defaults in `values.yaml`. The chart must resolve `selectedModel` by splitting at the **first** `/`: the first part is the provider name and the remainder is the model ID, which can itself contain slashes. Find the matching entry in `providers` to configure the client with its API base, optional key, and type. Only generate AI-related configuration when `ai.enabled` is `true`.

{% hint style="warning" %}
Injected credentials become part of the Helm values used to launch the service. Treat them as sensitive, including when inspecting or sharing generated values.
{% endhint %}

## Update an earlier AI configuration

Earlier development versions used a different configuration and launch context:

- Replace `DISABLE_AI` with `disable` inside `AI`, retaining `providers: []` if no providers are configured.
- Replace the top-level gateway object or array with `{ providers: [...] }`.
- Replace `URL` with the full `apiBase`, `provider` with `providerType`, and gateway authentication options with `authentification`. Provider identity now uses `name`, not `id`.
- Replace `accountCreation` help with `documentation` and its links.
- Update charts using `ai.activeProvider` to consume `ai.defaultModel`, `ai.models`, and `ai.providers`. There is no separate active-provider object.

If previously saved user settings cannot be read, the AI tab offers a reset. Resetting deletes saved custom providers, user-supplied API keys, and model selections; users must configure them again.
