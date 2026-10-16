---
icon: microchip-ai
---

# AI Integration

The main usecase of Onyxia is to launch IDE environement like VSCode, Jupyter or RStudio.\
We want to enable users to uses coding assistant like Claude Code or Codex in thoses environements.

For organizatino that would self host open weight LLMs models within their organization through [Open WebUI](https://openwebui.com/), onyxia is able to automatically exchange OpenID Connext token for AI API tokens.\
User can also setup their own tokens for popular AI providers: OpenAI, Antropic, Mistral and DeepSeek. Users tokens are [stored in Vault](readme/vault.md) if configured, in the browser storage otherwise.\
\
Uther can select what model they want as default and wich one they way to have available.&#x20;

<figure><img src="../.gitbook/assets/image (60).png" alt=""><figcaption></figcaption></figure>

Then services that supports it ([like the IDE provided in the Onyxia's default catalog](https://github.com/InseeFrLab/helm-charts-interactive-services/tree/main/charts)) will be able to provide Open source equivalents of Claude Code and the Codex VS Code extention, like for example respectively, Open Code and Continue. And use the model of their choosing with it.&#x20;

<figure><img src="../.gitbook/assets/image (61).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (63).png" alt=""><figcaption></figcaption></figure>

<details>

<summary>Understanding the boundary of what Onyxia does</summary>

At this stage of the developement of thse project, onyxia does not do anything with the AI API directly, it only makes thoses creedentials available to [the XOnyxia launcher context (`xOnyxia.ai`) ](catalog-of-services/custom-catalogs/onyxia-extension.md)context so that they can be injected in the services.   &#x20;

It's the responsability of the services to levrage thoses data to setup tools into the services to levrage those AI APIs credentials.  \
The services provided in the [catalog of interactive services](https://github.com/InseeFrLab/helm-charts-interactive-services) provided by default do it by setting up Continue and Open Code.

</details>

## API Reference

AI integration configuration is configured in the Region:

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

The AI configuration object respect this shape: &#x20;

{% code title="" %}
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

### Disabling all AI feature

By default user can go in their account and configure their API keys. If you want to disalow this:

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

### Allowing only one said provider

If you want, for example, that your user be allowed to use mistral but only mistral (Not OpenAI, Antropic or DeepSeek) you can do: &#x20;

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

### Self hosted LLMs

If your organization self host open weight LLMs model though Open WebUI, Onyxia is able to exchange the OpenID Connect token for a AI API token.  \
This way, the user don't even have to go in their acount and provide their API keys, they can simply launch a service and get Continue and Open Code readily available in their services.  \
\
For example, for https://datalab.my-domain.net Onyxia instance is integrated with our instance of Open WebUI https://llm.lab.my-domain.net\
\
The Onyxia configuration would look like:&#x20;

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
            en: "Use the models of your organization in your services. [Read more about this feature](https://www.$DOMAIN/document?path=SSPCloud%E2%90%A3Documentation%E2%80%BAUsing%E2%90%A3the%E2%90%A3Datalab%E2%80%BAUsing%E2%90%A3AI%E2%90%A3models%E2%90%A3in%E2%90%A3the%E2%90%A3Datalab)",
            fr: "Utilisez les modèles de votre organisation dans vos services. [En savoir plus sur cette fonctionnalité](https://www.$DOMAIN/document?path=SSPCloud%E2%90%A3Documentation%E2%80%BAUsing%E2%90%A3the%E2%90%A3Datalab%E2%80%BAUsing%E2%90%A3AI%E2%90%A3models%E2%90%A3in%E2%90%A3the%E2%90%A3Datalab)"
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
            }
          ]
        }
        }
```
{% endcode %}

This is what we would get in the UI: &#x20;

<figure><img src="../.gitbook/assets/image (64).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (65).png" alt=""><figcaption></figcaption></figure>

#### Configuring Open WebUI

{% hint style="info" %}
The onyxia-ops repo contain reference configuration:

Repo: [https://github.com/InseeFrLab/onyxia-ops](https://github.com/InseeFrLab/onyxia-ops)

Open WebUI Configuration: [https://github.com/InseeFrLab/onyxia-ops/blob/main/apps/openwebui/values.yaml](https://github.com/InseeFrLab/onyxia-ops/blob/main/apps/openwebui/values.yaml)\
Onyxia Configuration: [https://github.com/InseeFrLab/onyxia-ops/blob/024057d1be868b2e7f1b4b11584d673806d9a8f4/apps/onyxia/values-kitchen-sink.yaml#L1001-L1057](https://github.com/InseeFrLab/onyxia-ops/blob/024057d1be868b2e7f1b4b11584d673806d9a8f4/apps/onyxia/values-kitchen-sink.yaml#L1001-L1057)\
Keycloak realm configuration: [https://github.com/InseeFrLab/onyxia-ops/blob/main/apps/keycloak/realm-export.json](https://github.com/InseeFrLab/onyxia-ops/blob/main/apps/keycloak/realm-export.json) - More instruction on how to setup Keycloak here: [user-authentication.md](readme/user-authentication.md "mention")<br>
{% endhint %}



Let's see how to make the token exchange work, assuming here you're using keycloak:\
\
Assume your Open WebUI instance is deployed at https://llm.lab.my-domain.net\
\
Keycloak configuration:<br>

Server: https://auth.lab.my-domain.net, realm: datalab\
\
\---

Client: openwebui

Confidential: true (there is a client secret)

Valid redirect URI: https://llm.lab.my-domain.net/oauth/oidc/callback\
\
\---\
\
Client: onyxia\
\
Confidential: false\
\
Audience includes "openwebui" - This means that the Access Token issued to the client "onyxia" are intended to be used (also) with your Open WebUI instance. Concreatly this means that the "aud" claim of the access token issued to onyxia should look like "aud": \["onyxia-api", "minio", "vault", "openwebui"]\
\
Valid Redirect URI: https://datalab.my-domain.net/\
\
\---\
\
This are the more relevent configurations of open WebUI, for a complete reference check: [https://github.com/InseeFrLab/onyxia-ops/blob/main/apps/openwebui/values.yaml](https://github.com/InseeFrLab/onyxia-ops/blob/main/apps/openwebui/values.yaml)

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