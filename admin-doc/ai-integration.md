---
icon: microchip-ai
---

# AI Integration

Onyxia launches IDE environments such as VS Code, JupyterLab, and RStudio. Its AI integration lets users configure access to AI models for coding assistants in those environments.

Organizations that serve models through [Open WebUI](https://openwebui.com/) can configure Onyxia to exchange OpenID Connect (OIDC) access tokens for Open WebUI API tokens automatically. This includes organizations that host open weight large language models (LLMs) on their own infrastructure.

Users can also supply their own API keys for providers such as OpenAI, Anthropic, Mistral, and DeepSeek. These user-provided keys are [stored in Vault](readme/vault.md) when Vault is configured, or in browser storage otherwise.

In **Account > AI**, users can choose which models to make available in their services and select a default model.

<figure><img src="../.gitbook/assets/image (60).png" alt=""><figcaption></figcaption></figure>

Compatible services, such as the [IDEs in Onyxia's default catalog](https://github.com/InseeFrLab/helm-charts-interactive-services/tree/main/charts), can use these settings to configure coding assistants such as OpenCode and Continue with the selected providers and models.

<figure><img src="../.gitbook/assets/image (61).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (63).png" alt=""><figcaption></figcaption></figure>

<details>

<summary>Understanding what Onyxia does</summary>

Onyxia manages provider configuration, obtains credentials, and retrieves model lists. It does not send prompts to models or run inference. It exposes the provider credentials and selected models through the [launcher context (`xOnyxia.ai`)](catalog-of-services/custom-catalogs/onyxia-extension.md), so charts can inject them into launched services.

Each service's chart is responsible for using this configuration to set up its coding tools. Compatible charts in the default [catalog of interactive services](https://github.com/InseeFrLab/helm-charts-interactive-services) do this for Continue and OpenCode.

</details>

## API Reference

Configure AI integration under `onyxia.api.regions[0].AI` in your Helm values. The Helm chart uses the first region's `AI` configuration for the web application, unless `onyxia.web.env.AI` is explicitly set. The examples on this page show only the relevant settings; merge them into your existing region configuration.

{% code title="apps/onyxia/values.yaml" %}

```yaml
onyxia:
  api:
    regions:
      - AI:
          providers: []
```

{% endcode %}

AI integration is available to signed-in users. By default, no providers are provisioned by the administrator, and users can add their own. When you supply an `AI` object, `providers` is required, including when AI integration is disabled.

The `AI` configuration object has the following shape. `LocalizedString` means a string or an object mapping supported language codes to strings, such as `{ en: "...", fr: "..." }`. `ArrayOrNot<T>` means `T | T[]`. The property names `authentification` and `obtentionMethod` below match the configuration API.

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
            links?: { label: LocalizedString; url: string }[];
        };
        /**
         * Image displayed on the provider card: one absolute URL for both themes or
         * `{ light: "https://...", dark: "https://..." }`. Omit to show no logo;
         * admin-provided providers do not fall back to the protocol's built-in logo.
         */
        logoUrl?: string | { light: string; dark: string };
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
                  /**
                   * Optional overrides for this provider's OIDC settings.
                   * Omitted fields inherit Onyxia's login settings.
                   */
                  oidcConfiguration?: {
                      issuerURI?: string;
                      clientID?: string;
                      extraQueryParams?: string; // URL query string
                      scope?: string; // Space-separated scopes
                      idleSessionLifetimeInSeconds?: number | string;
                  };
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

## Example Configurations

### Disabling all AI features

To hide **Account > AI** and provide an empty, disabled `xOnyxia.ai` launch context, set `disable` to `true`:

{% code title="apps/onyxia/values.yaml" %}

```yaml
onyxia:
  api:
    regions:
      - AI:
          disable: true
          providers: []
```

{% endcode %}

### Provisioning a single provider

To provision Mistral and prevent users from adding other providers through **Account > AI**, use the configuration below. Users still need to enter their own Mistral API key.

`disallowUserToAddProviders` prevents new personal providers from being added. Previously saved personal providers remain available, so this setting does not enforce an exclusive provider list.

{% code title="apps/onyxia/values.yaml" %}

```yaml
onyxia:
  api:
    regions:
      - AI:
          disable: false
          disallowUserToAddProviders: true
          providers:
            - name: "Mistral"
              providerType: "mistral"
              apiBase: "https://api.mistral.ai/v1"
              authentification:
                type: "api-key"
                obtentionMethod: "user-provided"
              # Optional: offer only these models in Onyxia.
              models:
                - "mistral-medium-3-5"
                - "mistral-small-2603"
                - "mistral-large-4"
```

{% endcode %}

Omit `models` to offer the models returned by the provider's `/models` endpoint. An explicit list controls which models Onyxia offers to services; it does not change the permissions associated with the user's API key. Use model IDs supported by your provider and coding tools.

### Self-hosted LLMs

If your organization serves models through Open WebUI, Onyxia can exchange the user's OIDC access token for an Open WebUI API token. Users can then launch compatible services with Continue and OpenCode configured, without entering an API key in Onyxia.

The following example connects Onyxia at `https://datalab.my-domain.net` to Open WebUI at `https://llm.lab.my-domain.net`, using the Keycloak realm `datalab`. Both applications use the same OIDC issuer.

Replace the example domain names, documentation links, and logo URL with your own values.

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
    regions:
      - AI:
          disable: false
          disallowUserToAddProviders: false
          description:
            en: >-
              Use your organization's models in your services.
              [Read more about this feature](https://www.my-domain.net/ai-models).
            fr: >-
              Utilisez les modèles de votre organisation dans vos services.
              [En savoir plus sur cette fonctionnalité](https://www.my-domain.net/ai-models).
          providers:
            - name: "Organization LLM"
              providerType: "openai-compatible"
              apiBase: "https://llm.lab.my-domain.net/api"
              authentification:
                type: "api-key"
                obtentionMethod: "open-webui-oidc-token-exchange"
              logoUrl: "https://datalab.my-domain.net/custom-resources/assets/openwebui.png"
              documentation:
                mainText:
                  en: >-
                    Access your organization's models through Open WebUI
                    and its OpenAI-compatible API.
                  fr: >-
                    Accédez aux modèles de votre organisation via Open WebUI
                    et son API compatible OpenAI.
                links:
                  - label:
                      en: "API documentation"
                      fr: "Documentation de l'API"
                    url: "https://docs.openwebui.com/reference/api-endpoints/"
                  - label:
                      en: "Models documentation"
                      fr: "Documentation des modèles"
                    url: "https://llm.lab.my-domain.net/workspace/models"
```

{% endcode %}

The screenshots below illustrate how the provider and its documentation appear in **Account > AI**:

<figure><img src="../.gitbook/assets/image (64).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (65).png" alt=""><figcaption></figcaption></figure>

#### Configuring Open WebUI

{% hint style="info" %}
The [onyxia-ops repository](https://github.com/InseeFrLab/onyxia-ops) contains reference configurations:

- [Open WebUI configuration](https://github.com/InseeFrLab/onyxia-ops/blob/main/apps/openwebui/values.yaml)
- [Onyxia AI configuration](https://github.com/InseeFrLab/onyxia-ops/blob/024057d1be868b2e7f1b4b11584d673806d9a8f4/apps/onyxia/values-kitchen-sink.yaml#L1001-L1057)
- [Keycloak realm configuration](https://github.com/InseeFrLab/onyxia-ops/blob/main/apps/keycloak/realm-export.json)

For Keycloak setup instructions, see [User authentication](readme/user-authentication.md).
{% endhint %}

The configuration below assumes:

- Open WebUI is deployed at `https://llm.lab.my-domain.net`.
- Keycloak is deployed at `https://auth.lab.my-domain.net`, with a realm named `datalab`.

In the `datalab` realm, configure the **`openwebui` client**:

- **Client authentication:** On (a confidential client with a client secret).
- **Valid redirect URI:** `https://llm.lab.my-domain.net/oauth/oidc/callback`.

Configure the **`onyxia` client**:

- **Client authentication:** Off (a public client).
- **Valid redirect URI:** `https://datalab.my-domain.net/`.
- **Access-token audience:** include `openwebui`, alongside any other resource servers used by Onyxia.

For example, an access token issued to `onyxia` may contain these claims:

```json
{
  "aud": ["onyxia-api", "minio", "vault", "openwebui"],
  "azp": "onyxia"
}
```

The audience (`aud`) identifies the intended resource servers. Open WebUI's `OAUTH_TOKEN_EXCHANGE_TRUSTED_CLIENT_IDS` setting checks the issuing client through token introspection; set it to `onyxia` for this example. See the [Open WebUI token exchange configuration](https://docs.openwebui.com/reference/env-configuration/#oauth_token_exchange_trusted_client_ids).

Apply the following Open WebUI settings. Replace `<openwebui-client-secret>` with the secret of the Keycloak `openwebui` client. For a complete deployment example, see the [reference values file](https://github.com/InseeFrLab/onyxia-ops/blob/main/apps/openwebui/values.yaml).

{% code title="apps/openwebui/values.yaml" %}

```yaml
open-webui:
  extraEnvVars:
    - name: OAUTH_CLIENT_ID
      value: "openwebui"
    - name: OAUTH_CLIENT_SECRET
      value: "<openwebui-client-secret>"
    - name: OPENID_PROVIDER_URL
      value: "https://auth.lab.my-domain.net/realms/datalab/.well-known/openid-configuration"
    - name: OPENID_REDIRECT_URI
      value: "https://llm.lab.my-domain.net/oauth/oidc/callback"
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

Users must sign in to Open WebUI at least once to create their account before using token exchange. With `ENABLE_OAUTH_SIGNUP` enabled, they can do this through Keycloak. See the [Open WebUI token exchange requirements](https://docs.openwebui.com/reference/env-configuration/#enable_oauth_token_exchange).
