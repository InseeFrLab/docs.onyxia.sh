---
icon: microchip-ai
---

# AI Integration

Onyxia's main use case is launching IDE environments such as VS Code, Jupyter, and RStudio.

Onyxia's AI integration makes users' API credentials available to these environments at launch time, so services can configure open source coding assistants such as [OpenCode](https://opencode.ai/) and [Continue](https://docs.continue.dev/). These tools offer functionality similar to Claude Code and the Codex VS Code extension, respectively. Unlike their proprietary counterparts, they are not tied to a specific provider and can work with different LLM APIs, giving users a choice of providers and models.

In the Onyxia UI, users can set up their own API keys for popular AI providers: OpenAI, Anthropic, Mistral, and DeepSeek. Users' API keys are [stored in Vault](readme/vault.md) when Vault is configured, or in browser storage otherwise.

For organizations that host open weight LLMs on their own infrastructure through [Open WebUI](https://openwebui.com/), Onyxia automatically obtains the API key when token exchange is configured, with no manual action required from the user.

Users can select which models to make available in their services and choose one as the default.

<figure><img src="../.gitbook/assets/image (60).png" alt=""><figcaption></figcaption></figure>

Services that support this feature ([such as the IDEs in Onyxia's default catalog](https://github.com/InseeFrLab/helm-charts-interactive-services/tree/main/charts)) can then provide coding assistant tools, preconfigured and ready to use with the user's chosen LLM.

<figure><img src="../.gitbook/assets/image (61).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (63).png" alt=""><figcaption></figcaption></figure>

<details>

<summary>Understanding what Onyxia does</summary>

Concretely, Onyxia provides a UI in the Account > AI section that lets users bring their own API keys, choose which models to make available, and select a default model.

For organizations with an Open WebUI instance, it can be configured to automatically exchange an OpenID Connect access token for an Open WebUI API key, with no manual action required from the user.

Onyxia then makes all these credentials available through [the XOnyxia launcher context (`xOnyxia.ai`)](catalog-of-services/custom-catalogs/onyxia-extension.md). Services are responsible for using this information to configure tools such as OpenCode and Continue so that they are ready to use without requiring any manual configuration from the user.

The services in the [catalog of interactive services](https://github.com/InseeFrLab/helm-charts-interactive-services), one of the default catalogs included with Onyxia, already implement this approach.

In the future, Onyxia might use these AI credentials to provide AI features directly in its UI, such as an assistant to help configure services for a specific use case or "Explain with AI" buttons for commands or logs displayed by Onyxia.
However, these features are not currently implemented. Onyxia uses the APIs to obtain credentials and list available models; the services are responsible for sending prompts to the models.

</details>

## API Reference

AI integration settings are defined in the region configuration:

{% code title="apps/onyxia/values.yaml" %}

```yaml
onyxia:
  api:
    regions:
      - {
          // The AI configuration object
          IA: {
            providers: [...]
          }
        }
```

{% endcode %}

The AI configuration object has the following shape:

{% code title="AIConfigsObject" %}

```typescript
type AIConfigsObject = {
    /**
     * Hides Account > AI and supplies an empty, disabled `ai` launch context.
     * @default false
     */
    disable?: boolean;
    /**
     * Prevents users from adding their own providers. Admin-provided providers can
     * still be configured by users, and previously saved personal providers remain
     * available; this does not restrict the provider list to the ones declared here.
     * @default false
     */
    disallowUserToAddProviders?: boolean;
    /**
     * Markdown displayed below the "AI Providers" heading in Account > AI. Accepts
     * a string or translations such as `{ en: "...", fr: "..." }`. Omit to show no
     * introduction.
     */
    description?: LocalizedString;
    /**
     * Providers provisioned by the administrator, as a single object or an array.
     * Use `[]` to provision none. They appear before personal providers; this order
     * also determines the fallback default model when the user has not chosen one.
     */
    providers: ArrayOrNot<AIConfigsObject.Provider>;
};

namespace AIConfigsObject {
    /**
     * An admin-provided connection whose name, protocol and API base users cannot
     * edit. Users can select its models and supply an API key when requested.
     */
    export type Provider = {
        /**
         * Display name and persistent provider identifier. Trimmed on parsing, must
         * be unique among configured providers, and cannot contain `/` because launch
         * model identifiers use `<providerName>/<modelId>`.
         *
         * Keep this stable: saved API keys, excluded models and the user's default
         * model refer to it by name. A personal provider with the same name remains
         * visible for renaming or deletion but is excluded from the launch context.
         */
        name: string;
        /**
         * Protocol used to list models and exposed to charts as `ai.providers[].type`.
         * `anthropic` uses Anthropic headers; all other types use OpenAI-compatible
         * model listing with Bearer authentication when a key is available.
         */
        providerType: "openai-compatible" | "openai" | "anthropic" | "mistral" | "deepseek";
        /**
         * Absolute API root, including any required prefix such as `/v1`. Trailing
         * slashes are removed, and model discovery requests `<apiBase>/models`
         * directly from the browser. The same root is passed to launched services.
         */
        apiBase: string;
        /** Help displayed in this provider's management dialog. Omit to hide it. */
        documentation?: {
            /**
             * Plain text, either a string or translations such as
             * `{ en: "...", fr: "..." }`.
             */
            mainText: LocalizedString;
            /**
             * Links opened in a new tab, with localized labels and absolute URLs.
             * Defaults to an empty list.
             */
            links?: AiConfig.Documentation.Link[];
        };
        /**
         * Image displayed on the provider card: one absolute URL for both themes or
         * `{ light: "https://...", dark: "https://..." }`. Omit to show no logo;
         * admin-provided providers do not fall back to the protocol's built-in logo.
         */
        logoUrl?: string | { light: string; dark: string; };
        /**
         * How the provider's credentials are obtained. An authenticated provider
         * with at least one selected model is included in the launch context along
         * with its API key; `none` providers are included without a key.
         */
        authentification:
            | {
                  /** No credentials required; model requests omit the API key. */
                  type: "none";
              }
            | {
                  type: "api-key";
                  /** The user enters their own API key in the provider's management dialog. */
                  obtentionMethod: "user-provided";
              }
            | {
                  type: "api-key";
                  /**
                   * Exchanges an OIDC access token for an Open WebUI API token via
                   * `POST <apiBase>/v1/auths/oauth/oidc/token/exchange`. The resulting
                   * token is refreshed before subsequent service launches.
                   */
                  obtentionMethod: "open-webui-oidc-token-exchange";
              };
        /**
         * Explicit model IDs offered instead of the discovered list. Duplicates are
         * removed, preserving order. Omit to use the provider's `/models` response;
         * `[]` deliberately offers no models and excludes this provider from launches.
         *
         * Model discovery is still attempted to report connection status, but an
         * explicit list remains selectable if that request fails (for example, when
         * only launched services can reach the provider). Authentication must still
         * succeed. All offered models are selected by default unless the user has
         * excluded them; the first selected model is a fallback for the default.
         */
        models?: string[];
    };
}
```

{% endcode %}

## Example Configurations

### Disabling all AI features

By default, users can configure their API keys in their account settings. To disable this:

{% code title="apps/onyxia/values.yaml" %}

```yaml
onyxia:
  api:
    regions:
      - {
          IA: {
            disable: true,
            providers: []
          }
        }
```

{% endcode %}

### Allowing only a specific provider

For example, to allow your users to use only Mistral (and not OpenAI, Anthropic, or DeepSeek), use the following configuration:

{% code title="apps/onyxia/values.yaml" %}

```yaml
onyxia:
  api:
    regions:
      - {
          IA: {
            disable: false,
            disallowUserToAddProviders: true,
            providers: [
              {
                name: "Mistral",
                providerType: "mistral",
                apiBase: "https://api.mistral.ai/v1",
                authentification: {
                  type: "api-key",
                  obtentionMethod: "user-provided"
                },
                # If you want to restrict usage to specific models from the provider:
                models: ["mistral-medium-3-5", "mistral-small-2603", "mistral-large-4"]
              }
            ]
          }
        }
```

{% endcode %}

### Self-hosted LLMs

If your organization hosts open weight LLMs through [Open WebUI](https://openwebui.com/), Onyxia can exchange the OpenID Connect token for an AI API token.
Users do not need to go to their account settings and provide API keys. They can simply launch a service with [Continue](https://docs.continue.dev/) and [OpenCode](https://opencode.ai/) ready to use.

For example, an Onyxia instance at `https://datalab.my-domain.net` can be integrated with an [Open WebUI](https://openwebui.com/) instance at `https://llm.lab.my-domain.net`

The Onyxia configuration would look like this:

{% code title="apps/onyxia/values.yaml" %}

```yaml
onyxia:
  ingress:
    enabled: true
    hosts:
      - host: datalab.my-domain.net
  api:
    env:
      authentication.mode: "openidconnect"
      oidc.issuer-uri: "https://auth.lab.my-domain.net/realms/datalab"
      oidc.clientID: "onyxia"
      # .//
    regions:
      - {
          AI: {
            disable: false,
            disallowUserToAddProviders: false,
            description: {
              en: "Choose which AI models to use with [OpenCode](https://opencode.ai/) and [Continue](https://docs.continue.dev/) in your services. [Read more about this feature](https://www.sspcloud.fr/document?path=SSPCloud%E2%90%A3Documentation%E2%80%BAUsing%E2%90%A3the%E2%90%A3Datalab%E2%80%BAUsing%E2%90%A3AI%E2%90%A3models%E2%90%A3in%E2%90%A3the%E2%90%A3Datalab)",
              fr: "Choisissez les modèles d'IA à utiliser avec [OpenCode](https://opencode.ai/) et [Continue](https://docs.continue.dev/) dans vos services. [En savoir plus sur cette fonctionnalité](https://www.sspcloud.fr/document?path=SSPCloud%E2%90%A3Documentation%E2%80%BAUsing%E2%90%A3the%E2%90%A3Datalab%E2%80%BAUsing%E2%90%A3AI%E2%90%A3models%E2%90%A3in%E2%90%A3the%E2%90%A3Datalab)"
            },
            providers: [
              {
                name: "SSPCloud LLM",
                providerType: "openai-compatible",
                apiBase: "https://llm.lab.my-domain.net/api",
                authentification: {
                  type: "api-key",
                  obtentionMethod: "open-webui-oidc-token-exchange"
                },
                logoUrl: "%PUBLIC_URL%/custom-resources/assets/openwebui.png",
                documentation: {
                  mainText: {
                    en: "SSPCloud LLM uses Open WebUI to give you access to AI models and its OpenAI-compatible API.",
                    fr: "SSPCloud LLM utilise Open WebUI pour vous donner accès à des modèles d'IA et à son API compatible OpenAI."
                  },
                  links: [
                    {
                      label: { en: "API documentation", fr: "Documentation de l'API" },
                      url: "https://docs.openwebui.com/reference/api-endpoints/"
                    },
                    {
                      label: { en: "Models documentation", fr: "Documentation des modèles" },
                      url: "https://llm.lab.my-domain.net/workspace/models"
                    },
                    {
                      label: { en: "Models configuration (vLLM)", fr: "Configuration des modèles (vLLM)" },
                      url: "https://github.com/InseeFrLab/sspcloud-llm"
                    }
                  ]
                }
              }
            ]
          }
        }
```

{% endcode %}

This is how it would appear in the UI:

<figure><img src="../.gitbook/assets/image (64).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (65).png" alt=""><figcaption></figcaption></figure>

#### Configuring Open WebUI

{% hint style="info" %}
The onyxia-ops repository contains reference configurations:

- **Repo:** [onyxia-ops](https://github.com/InseeFrLab/onyxia-ops)
- **[Open WebUI](https://openwebui.com/) configuration:** [apps/openwebui/values.yaml](https://github.com/InseeFrLab/onyxia-ops/blob/main/apps/openwebui/values.yaml)
- **Onyxia configuration:** [apps/onyxia/values-kitchen-sink.yaml](https://github.com/InseeFrLab/onyxia-ops/blob/024057d1be868b2e7f1b4b11584d673806d9a8f4/apps/onyxia/values-kitchen-sink.yaml#L1001-L1057)
- **Keycloak realm configuration:** [apps/keycloak/realm-export.json](https://github.com/InseeFrLab/onyxia-ops/blob/main/apps/keycloak/realm-export.json)

For more instructions on setting up Keycloak, see [User authentication](readme/user-authentication.md "mention")
{% endhint %}

Let's see how to configure token exchange, assuming you are using Keycloak:

Assume your [Open WebUI](https://openwebui.com/) instance is deployed at `https://llm.lab.my-domain.net`

**Keycloak configuration:**

**Server:** `https://auth.lab.my-domain.net`, **realm:** `datalab`

{% stepper %}
{% step %}
#### Client: `openwebui`

- **Confidential:** `true` (there is a client secret)
- **Valid redirect URI:** `https://llm.lab.my-domain.net/oauth/oidc/callback`
{% endstep %}

{% step %}
#### Client: `onyxia`

- **Confidential:** `false`
- **Audience includes** `"openwebui"`
- **Valid redirect URI:** `https://datalab.my-domain.net/`

This means that access tokens issued to the `"onyxia"` client are also intended for your [Open WebUI](https://openwebui.com/) instance. Specifically, the `"aud"` claim of an access token issued to `"onyxia"` should look like `"aud": ["onyxia-api", "minio", "vault", "openwebui"]`
{% endstep %}
{% endstepper %}

These are the most relevant [Open WebUI](https://openwebui.com/) settings. For a complete reference, see: [Open WebUI configuration](https://github.com/InseeFrLab/onyxia-ops/blob/main/apps/openwebui/values.yaml)

{% code title="apps/openwebui/values.yaml" %}

```yaml
open-webui:
  extraEnvVars:
    - name: OAUTH_CLIENT_ID
      value: openwebui
    - name: OAUTH_CLIENT_SECRET
      value: <TODO REPLACE>
    - name: OPENID_PROVIDER_URL
      value: https://auth.lab.$DOMAIN/realms/sspcloud/.well-known/openid-configuration
    - name: OPENID_REDIRECT_URI
      value: https://llm.lab.$DOMAIN/oauth/oidc/callback
    - name: ENABLE_OAUTH_TOKEN_EXCHANGE
      value: "true"
    - name: OAUTH_TOKEN_EXCHANGE_TRUSTED_CLIENT_IDS
      value: "onyxia"
    - name: ENABLE_OAUTH_SIGNUP
      value: "true"
    - name: OAUTH_MERGE_ACCOUNTS_BY_EMAIL
      value: "true"
    - name: ENABLE_LOGIN_FORM
      value: "false"
```

{% endcode %}
