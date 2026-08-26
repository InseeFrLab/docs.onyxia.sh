---
description: Select an AI gateway or connect your own AI provider for compatible Onyxia services.
icon: sparkles
---

# Configure AI providers

The **My account > AI** tab lets you choose the AI provider and model that compatible Onyxia services use when they start.

{% hint style="info" %}
The tab is visible by default for authenticated users unless your platform administrator has disabled AI integration. A service must also explicitly support Onyxia's AI configuration; selecting a provider does not add AI features to every catalog service.
{% endhint %}

<figure><img src="https://github.com/user-attachments/assets/7bb15d68-d531-4221-9f89-007d0fbb5920" alt="The AI tab showing a region gateway and a custom provider"><figcaption><p>Manage gateway credentials, providers, and model selections from your account.</p></figcaption></figure>

## Use your platform's AI gateway

When your administrator provides an AI gateway, Onyxia uses your login session to request a gateway token automatically.

1. Open **My account > AI**.
2. If Onyxia says that you do not have an account, open the gateway and sign in once. Return to Onyxia and select **Refresh credentials**.
3. Select a model from the gateway's model list.
4. If several providers are available, select **Set default provider** on the one your services should use.

You can copy the API base URL and token to configure a compatible client manually. Treat the token like a password. It can expire; use **Refresh credentials** to obtain a new one.

Onyxia remembers the selected model and default provider. The gateway token itself is obtained again from your OIDC session rather than stored as a long-lived credential.

## Add a custom provider

You can connect a provider for which you already have an API key:

1. Under **Custom AI providers**, select **Add a Custom AI Provider**.
2. Enter a name and choose the API protocol.
3. Check the API base URL and enter your API key.
4. Select **Test connection**. Onyxia calls the provider's `/models` endpoint and loads the models available to your key.
5. Select a model, optionally make the provider the default, and save it.

The supported protocols and default API base URLs are:

| Protocol | Default API base URL |
| --- | --- |
| OpenAI (native) | `https://api.openai.com/v1` |
| OpenAI-compatible | You must provide the URL. |
| Mistral (native) | `https://api.mistral.ai/v1` |
| Anthropic (native) | `https://api.anthropic.com/v1` |

Enter the base URL, not the full models or chat-completions endpoint. For example, use `https://api.openai.com/v1`, not `https://api.openai.com/v1/models`. Avoid a trailing slash because Onyxia appends `/models` when testing the connection.

{% hint style="warning" %}
Onyxia contacts custom providers directly from your browser. The provider must allow cross-origin requests from your Onyxia URL. A connection test can fail even with a valid key if the endpoint's CORS configuration blocks the request.
{% endhint %}

You can later change the model, edit the provider, make it the default, or delete it. Changing the API base URL or API key requires a new successful connection test before the provider can be saved.

## Where custom credentials are stored

Custom provider settings include the API key:

* If your Onyxia region provides Vault, they are saved with your Vault-backed Onyxia user configuration.
* Without Vault, they are saved in the browser's local storage and are only available in that browser profile.

Deleting a custom provider removes it from this saved configuration. On a shared computer, sign out and follow your organization's browser-data policy.

## Use the provider in a service

After choosing a default provider and model, launch an AI-compatible service from the catalog. Onyxia injects the provider protocol, API base URL, credential, and selected model into the chart when it builds the Helm values.

The exact behavior inside the service depends on its chart. Consult the service's README to find the relevant environment variables, client configuration, and supported capabilities.

## Troubleshooting

| Problem | What to check |
| --- | --- |
| The AI tab is missing | Your administrator must enable the feature, and you must be signed in. |
| The gateway says that you have no account | Open the gateway, sign in once, return to Onyxia, and refresh the credentials. |
| Gateway models do not load | Refresh the credentials. If the error continues, contact the platform administrator. |
| A custom provider test fails | Verify the protocol, base URL, API key, `/models` support, and browser CORS policy. |
| The wrong provider is used by a service | Make the intended provider the default before launching the service. |
| A service receives no AI configuration | The chart must explicitly support Onyxia AI integration; check its README or contact its maintainer. |

Platform administrators and chart maintainers can find the complete setup in [AI integration](../admin-doc/ai-configuration.md).
