---
description: Configure the Git forges your users connect to, how the services get their credentials, and how charts consume them.
icon: code-branch
---

# Git integration

Onyxia lets each user connect their Git accounts (GitLab, Forgejo, GitHub, Bitbucket Cloud) in **My account > Git**. When a service is launched, Onyxia hands over, for every connected forge, a commit identity and credentials, so that the service can clone and push without the user pasting a token in the launcher.

Administrators declare the forges of their organization and choose, per forge, how Onyxia accesses it and what the services receive: the user's personal access token, a short-lived token created at launch, or an OAuth authorization. Users can also add forges of their own with a personal access token, unless the administrator disables it.

{% hint style="info" %}
This replaces the former single "GitHub personal access token" field of the user profile. A token saved there is migrated, once, to the `github.com` provider. A chart must use the [`git.providers` x-onyxia context](#inject-the-credentials-into-a-service) to configure several forges; the former `git.name`, `git.email` and `git.token` fields are still filled for existing charts.
{% endhint %}

## Enable or disable the integration

Configure the integration through the `GIT_PROVIDERS` environment variable of Onyxia Web. The value is a JSON5 **object**. In Helm values, use a YAML literal block (`|`) to pass it as a string.

An unset or empty `GIT_PROVIDERS` enables the integration with no administrator-configured provider: users add their own. To disable it entirely:

{% code title="apps/onyxia/values.yaml" %}
```yaml
onyxia:
  web:
    env:
      GIT_PROVIDERS: |
        { providers: [], disallowUserToAddProviders: true }
```
{% endcode %}

The **Git** tab is then empty, and services receive the Onyxia identity (name and email of the user's account) without credentials.

| Property | Default | Description |
| --- | --- | --- |
| `providers` | Required for a nonempty configuration | An array of provider objects. Use `[]` for none. |
| `disallowUserToAddProviders` | `false` | Hide the option to add custom providers. Providers the users already added are then ignored. |
| `description` | Unset | Introductory Markdown below the **Git profiles** heading; a string or a localized object such as `{ en: "...", fr: "..." }`. Use it to link to your own documentation. |

{% hint style="warning" %}
The `GIT_PROVIDERS` configuration is exposed to the browser. Do not put client secrets, tokens or other secrets in it. Every OAuth application referenced here is a public client, without secret.
{% endhint %}

## Configure providers

This example offers an organization GitLab, where Onyxia signs in with OAuth and creates a token for every service, and `github.com`, where each user supplies a personal access token:

{% code title="apps/onyxia/values.yaml" %}
```yaml
onyxia:
  web:
    env:
      GIT_PROVIDERS: |
        {
          description: {
            en: "Connect your Git accounts to clone and push from your services. [Read more](https://example.com/git)",
            fr: "Connectez vos comptes Git pour cloner et pousser depuis vos services. [En savoir plus](https://example.com/git)"
          },
          providers: [
            {
              type: "gitlab",
              url: "https://git.example.com",
              displayName: "Organization GitLab",
              authentication: {
                ui: { type: "oauth", clientId: "<application id of the Onyxia OAuth app>" },
                service: { type: "scoped-access-token", lifetimeDays: 30 }
              }
            },
            {
              type: "github",
              url: "https://github.com",
              authentication: {
                ui: { type: "personal-access-token" },
                service: { type: "personal-access-token" }
              }
            }
          ]
        }
```
{% endcode %}

### Provider properties

| Property | Required | Description |
| --- | --- | --- |
| `type` | Yes | `gitlab`, `forgejo`, `github` or `bitbucket`. `bitbucket` is Bitbucket Cloud only (`https://bitbucket.org`). |
| `url` | Yes | Base URL of the forge, for example `https://gitlab.com`. Onyxia removes trailing slashes. |
| `authentication` | Yes | `{ ui, service }`, see [Authentication](#authentication). |
| `displayName` | No | Defaults to the host of the `url`. Must be unique: the same `url` can be configured twice with distinct display names (for example with two authentications). |
| `logoUrl` | No | Image URL, or `{ light: "https://...", dark: "https://..." }`. Defaults to the logo of the forge type. |

The display name is the identity of the provider: tokens, sessions and launch configurations are saved under it. **Renaming a provider signs its users out.** It cannot start with `user:`, which is reserved for the providers users add themselves.

## Authentication

Each provider defines two authentications:

* `ui`: how Onyxia, in the browser, accesses the forge (identify the user, list repositories and branches).
  * `{ type: "oauth", clientId }`: the user signs in through a popup.
  * `{ type: "personal-access-token" }`: the user pastes a token in **My account > Git**.
* `service`: what the launched services receive to clone and push.
  * `{ type: "personal-access-token" }`: the user's token from the `ui` authentication. It is the only option when `ui` is `personal-access-token`: the token the user creates is the one handed over to the services.
  * `{ type: "scoped-access-token", lifetimeDays }`: Onyxia creates a dedicated token, named `onyxia-<service name>`, when the user clicks **Launch**. `lifetimeDays` defaults to `30`.
  * `{ type: "oauth", clientId }`: the service gets its own OAuth authorization, through an OAuth application distinct from Onyxia's.

### Supported combinations

| `ui` / `service` | GitLab | Forgejo | GitHub | Bitbucket Cloud |
| --- | :-: | :-: | :-: | :-: |
| `oauth` / `scoped-access-token` | ✅ | | | |
| `oauth` / `oauth` | ✅ | ✅ | | |
| `personal-access-token` / `personal-access-token` | ✅ | ✅ | ✅ | ✅ |

Onyxia rejects any other combination at startup with an explicit error:

* GitHub's and Bitbucket's token endpoints require a client secret, so Onyxia cannot sign in to them with OAuth. Use personal access tokens.
* With `ui: oauth`, there is no personal access token to hand over: `service` can't be `personal-access-token`.
* With `ui: personal-access-token`, the services get that same token: `service` must be `personal-access-token` too.
* Only GitLab lets Onyxia create tokens: `scoped-access-token` is GitLab only.

### Recommended setup for a GitLab instance

`oauth` / `scoped-access-token` gives every service its own token, limited to cloning and pushing, which expires on its own and can be revoked from GitLab without affecting the other services:

* Requires **GitLab 19.2 or later**, with fine-grained personal access tokens enabled.
* The token is created with the `download_code` and `push_code` permissions on the user's personal projects and on all the projects they are a member of.
* GitLab expires tokens at 00:00 UTC on their expiration date: the expiration advertised to the service is that midnight.

### OAuth applications

OAuth applications are **public** (no client secret, PKCE) with the redirect URI:

```
<onyxia url>/git/oauth/callback
```

| Forge | Scopes requested by Onyxia (`ui`) | Scopes requested for the service |
| --- | --- | --- |
| GitLab | `read_api` (`api` with `scoped-access-token`) | `read_repository`, `write_repository` |
| Forgejo | `read:user`, `read:repository` | `write:repository` |

For `service: { type: "oauth" }`, register a second application with its own `clientId`, also public, with the same redirect URI. Onyxia opens an authorization popup when the user clicks **Launch** and hands the refresh token over to the service. On GitLab, the service falls back on the [device flow](https://docs.gitlab.com/api/oauth2/#device-authorization-grant-flow) if the refresh token can no longer be used: a terminal of the service shows a link and a code. Forgejo has no device flow.

{% hint style="warning" %}
Forgejo doesn't enforce OAuth scopes yet: its OAuth tokens have every right of the user, as a personal access token created with every scope would.
{% endhint %}

### Cross-origin requests

Onyxia calls the forge's API directly from the user's browser. The forge must allow the Onyxia origin through CORS. GitLab, GitHub and Bitbucket Cloud do; a **Forgejo instance needs** in its `app.ini`:

```ini
[cors]
ENABLED = true
```

The same applies to the forges users add themselves.

## Providers added by the users

Unless `disallowUserToAddProviders` is `true`, users can add their own providers in **My account > Git** (`gitlab.com`, `github.com`, a Forgejo of their own, Bitbucket Cloud...). These always use `personal-access-token` / `personal-access-token`: the token is checked against the forge, then handed over to the services.

## Where credentials are stored

* Personal access tokens and the providers added by the user are saved in the user's Onyxia settings: in Vault when it is configured, otherwise in the browser's local storage.
* OAuth refresh tokens of the `ui` authentication are kept in the browser's local storage, per user. They are not cleared when the user signs out of Onyxia. Access tokens stay in memory.
* Credentials created at launch (scoped tokens, the service's OAuth refresh token) are **never** saved in the launcher's state: the form holds a placeholder, replaced only in the values sent to the API. Saved configurations and shared launch links contain no secret.

Disconnecting a provider in Onyxia revokes the session when the forge allows it; tokens created for running services stay valid until they expire or are revoked on the forge.

## Inject the credentials into a service

The launcher exposes the following [`x-onyxia`](catalog-of-services/custom-catalogs/onyxia-extension.md) context:

| Context path | Value |
| --- | --- |
| `git.enabled` | `true` when at least one Git provider is connected. |
| `git.providers` | The connected providers, see below. |
| `git.name`, `git.email` | **Deprecated.** Commit identity on the first connected provider, or the Onyxia account's. |
| `git.token` | **Deprecated.** Token of the first connected provider, possibly `undefined`. |
| `git.credentials_cache_duration` | **Deprecated.** Only meaningful for the former `cache` credential helper. |

Each entry of `git.providers` has the following shape:

```typescript
{
    id: string;      // The display name, or "user:<host>" for a provider added by the user
    url: string;     // Without trailing slash, e.g. "https://gitlab.com"
    user: {
        name: string;   // Commit identity on this forge (git config user.name / user.email)
        email: string;  // Falls back on the Onyxia account's when the forge doesn't share it
    };
    credentials:
        | {
              type: "token";
              username: string;  // Paired with the token for git over HTTPS
              token: string;
              expirationTime: number | undefined; // Unix time in ms, undefined: doesn't expire
          }
        | {
              type: "oauth";     // For git-credential-oauth
              clientId: string;
              scopes: string[];
              authorizationUrl: string;
              tokenUrl: string;
              deviceAuthorizationUrl: string | undefined; // Fallback when refreshing fails
              refreshToken: string;   // Obtained by Onyxia when the user clicks "Launch"
          }
        | undefined;
}
```

This JSON Schema fragment belongs under the root `properties` of a chart's `values.schema.json`. It passes the providers as a hidden value and lets the user pick repositories to clone:

{% code title="values.schema.json (fragment)" %}
```json
{
  "git": {
    "type": "object",
    "properties": {
      "enabled": {
        "type": "boolean",
        "default": false,
        "x-onyxia": { "overwriteDefaultWith": "{{git.enabled}}", "hidden": true }
      },
      "providers": {
        "type": "array",
        "default": [],
        "x-onyxia": { "overwriteDefaultWith": "{{git.providers}}", "hidden": true }
      },
      "repositories": {
        "type": "array",
        "title": "Repositories to clone",
        "default": [],
        "items": {
          "type": "object",
          "render": "gitRepository",
          "default": { "providerId": "", "url": "", "branch": "" },
          "properties": {
            "providerId": { "type": "string" },
            "url": { "type": "string" },
            "branch": { "type": "string" }
          }
        }
      }
    }
  }
}
```
{% endcode %}

`render: "gitRepository"` applies to an object `{ providerId, url, branch }`, alone or as the items of an array. The launcher renders a field to search the repositories of the connected providers, or to paste any HTTPS clone URL, with a branch selector. An empty `branch` means the default branch. Other properties of the object are kept as they are.

The [images-datascience](https://github.com/InseeFrLab/images-datascience) init script reads two environment variables:

* `GIT_PROVIDERS`: `git.providers` as JSON. For every provider, it sets the commit identity for the remotes of that forge (`includeIf.hasconfig:remote.*.url`) and configures the credential helper: `store` for a token, [git-credential-oauth](https://github.com/hickford/git-credential-oauth) for OAuth. The variable is then unset.
* `GIT_REPOSITORIES`: the repositories as JSON (`[{ url, branch }]`), cloned at startup with those credentials. Nothing is written in the clone URL.

{% code title="templates/secret-git.yaml" %}
```yaml
{{- if .Values.git.enabled }}
apiVersion: v1
kind: Secret
metadata:
  name: {{ .Release.Name }}-git
type: Opaque
stringData:
  GIT_PROVIDERS: {{ .Values.git.providers | toJson | quote }}
{{- end }}
```
{% endcode %}

{% code title="templates/statefulset.yaml (container env excerpt)" %}
```yaml
{{- if .Values.git.enabled }}
- name: GIT_PROVIDERS
  valueFrom:
    secretKeyRef:
      name: {{ .Release.Name }}-git
      key: GIT_PROVIDERS
{{- end }}
- name: GIT_REPOSITORIES
  value: {{ .Values.git.repositories | toJson | quote }}
```
{% endcode %}

{% hint style="warning" %}
Injected credentials become part of the Helm values used to launch the service. Store them in a Kubernetes `Secret` rather than in plain environment values, and treat them as sensitive when inspecting or sharing the generated values.
{% endhint %}

### Launch behavior

* With `scoped-access-token` or `service: oauth`, the credentials are created only when the user clicks **Launch**. If the service's OAuth authorization needs a popup, an automatic launch (`autoLaunch=true` link) stops on the form: the browser would block a popup opened without a click.
* If the credentials can't be obtained (popup blocked, authorization denied, expired session), nothing is launched and the user gets an explanation.
* Changes in **My account > Git** apply to subsequent launches. They don't update the credentials of a running service.
