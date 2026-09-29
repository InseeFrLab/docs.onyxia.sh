---
description: Configure AI providers, select available models, and choose a default for compatible Onyxia services.
icon: sparkles
---

# Configure AI providers

The **My account > AI** tab lets you manage the providers and models that compatible Onyxia services receive when they start.

{% hint style="info" %}
The tab is visible to signed-in users unless your administrator disables AI integration. A service must explicitly support Onyxia's AI configuration; selecting models does not add AI features to every catalog service.
{% endhint %}

## Use a provider supplied by your organization

Providers configured by your administrator are labeled **Provided by your organization**.

1. Open **My account > AI** and select **Manage** on a provider.
2. Read any documentation supplied by your organization.
3. If the provider requests an API key, enter your own key and save the changes. Other providers use your login session to obtain credentials automatically, or need no authentication.
4. Use **Test connection** to check access and load the model list. For providers using your login session, **Refresh credentials** obtains a new token.
5. Choose the models you want available in your services using **Selected models**.

For an OpenWebUI gateway, you may need to open the gateway and sign in once before returning to Onyxia and refreshing credentials. Follow your organization's instructions.

The management panel lets you copy the API base URL and any available credential to configure another compatible client. Treat credentials like passwords. Tokens obtained from your login session can expire.

## Add a custom provider

If your administrator allows it:

1. Select **Add a new custom AI provider**.
2. Enter a unique name without `/` and choose the **Provider API**.
3. Check the **API Base URL** and enter your API key. Leave the key empty only if the provider requires no authentication.
4. Select **Test connection** to discover the available models, then select the models you want to use.
5. Save the provider, then choose your default model at the top of the AI tab.

The supported provider types and suggested base URLs are:

| Provider API | Suggested API base URL |
| --- | --- |
| OpenAI (native) | `https://api.openai.com/v1` |
| OpenAI-compatible | Enter your provider's URL. |
| Mistral (native) | `https://api.mistral.ai/v1` |
| Anthropic (native) | `https://api.anthropic.com/v1` |
| DeepSeek | `https://api.deepseek.com` |

Enter the API base, including `/v1` or `/api` if required by your provider. Do not include the final `/models` or chat-completions path. Onyxia removes trailing slashes.

You can save a provider without a successful connection test and finish setup later. However, custom providers need a discovered model list and at least one selected model before they can be passed to a service.

{% hint style="info" %}
Onyxia tests providers directly from your browser. A valid API key is not enough if the endpoint is unreachable or its CORS policy blocks your Onyxia URL.
{% endhint %}

Use **Manage** to edit a custom provider, change its key, test the connection, or delete it. Organization-supplied names, API types, and base URLs are controlled by your administrator.

## Select models and choose a default

Each provider's **Selected models** control accepts multiple models. All available models are initially selected, including newly discovered models you have not previously excluded. Deselect all models to stop that provider from being passed to new services.

Use **Choose a default model** at the top of the tab to select a model across all providers. The selected value includes its provider name, for example `Organization AI/model-id`. If that model cannot be supplied at launch time, Onyxia uses the first available selected model.

Selections made from the main AI tab are saved automatically. If saving fails, your changes remain on the page and **Retry** lets you try again. Changes to provider details must be saved in the management panel.

## Where credentials are stored

Your custom providers, user-supplied API keys, model selections, and default model are stored with your Onyxia account settings:

- With Vault, they are saved in your Vault-backed user configuration.
- Without Vault, they are saved in the browser's local storage and are available only in that browser profile.

Tokens obtained automatically from an OpenWebUI login exchange are kept in the current session rather than saved as permanent API keys. Deleting a custom provider removes its saved configuration and key from Onyxia; it does not revoke the key at the provider.

## Use AI in a service

Launch a compatible service after configuring your providers and selecting models. Its chart receives the selected models, a default model, and the usable providers with their API settings. Some charts let you choose another selected model in the launch form.

The service's chart determines which AI features and clients are configured. Check its README for details. Changes in **My account > AI**, including refreshed credentials, apply to subsequent launches; they do not update an already running service.

## Troubleshooting

| Problem | What to check |
| --- | --- |
| The AI tab is missing | Sign in and check whether your administrator has disabled the feature. |
| The option to add a custom provider is missing | Your administrator may have disabled adding providers. |
| A provider shows **Setup required** | Open **Manage**, supply any required key, and test the connection. |
| An OpenWebUI exchange fails | Follow the gateway's documentation, sign in there if needed, and refresh credentials. Contact your administrator if the error persists. |
| The connection test fails | Check the API type, base URL, key, `/models` endpoint, and browser CORS policy. |
| Models remain selectable despite a connection error | Your administrator may have supplied a fixed list. The service still needs network access and any required credentials. |
| A provider is missing from a service | Select at least one model and check credentials. If a custom provider shares its name with an organization provider, rename it. The service's chart must support AI integration. |
| The wrong model is used | Check **Choose a default model** and any model selection in the launch form. An unavailable default falls back to the first usable selected model. |
| Changes cannot be saved | Keep the page open and use **Retry** once the connection is restored. |
| Saved AI configuration cannot be read | **Reset my AI configuration** clears custom providers, saved keys, and model selections. Use it only if you are ready to configure them again. |

Administrators and chart maintainers can find the setup details in [AI integration](../admin-doc/ai-configuration.md).
