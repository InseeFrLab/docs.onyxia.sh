---
description: Connect your Git accounts and clone your repositories in your Onyxia services.
icon: code-branch
---

# Connect your Git accounts

The **My account > Git** tab lets you connect your accounts on Git forges (GitLab, Forgejo, GitHub, Bitbucket Cloud). Once connected, your services can clone and push to your repositories, and the launcher lets you pick the repositories to clone at startup.

{% hint style="info" %}
A service must support Onyxia's Git integration to receive your credentials. Check its README. If the tab is empty and offers nothing to add, your administrator has disabled the integration.
{% endhint %}

## Use a provider supplied by your organization

Providers configured by your administrator are labeled **Provided by your organization**. Depending on the provider, either:

* select **Sign in**: a window of the forge opens, authorize Onyxia, and the card shows **Connected as** with your username. If nothing opens, allow popups for your Onyxia URL and try again; or
* select **Add access token**, follow **Generate a new token** to create one on the forge, paste it, then select **Confirm access token**. Onyxia checks the token with the forge before saving it.

When your session ends or a token expires, the card shows **Reconnect required** or **Sign in required**: select **Reconnect**, or replace the token. A card warns you when your token is about to expire.

**Sign out** disconnects the provider in Onyxia. To fully revoke Onyxia's access, also remove it from the authorized applications of your account on the forge.

## Add a custom Git profile

If your administrator allows it, you can add any other forge with a personal access token:

1. Select **Add a new custom Git profile**.
2. Choose the provider type (GitLab, Forgejo, GitHub or Bitbucket) and, for a self-hosted instance, enter its URL, for example `https://gitlab.example.com`.
3. Optionally, define a custom name.
4. Create a token on the forge with **Generate a token**, paste it, and select **Authenticate**.

The token needs read access to your profile and read and write access to your repositories:

| Forge | Token |
| --- | --- |
| GitLab | Personal access token with `read_api` and `write_repository` scopes. |
| GitHub | Classic token with the `repo` scope, or a fine-grained token with read and write access to *Contents*. |
| Forgejo | Access token with `read:user` and `write:repository`. The instance must allow cross-origin requests (`[cors] ENABLED = true`). |
| Bitbucket Cloud | API token with scopes: `read:user:bitbucket`, `read:workspace:bitbucket`, `read:repository:bitbucket` and `write:repository:bitbucket`. |

Use **Manage** to rename a custom profile, replace its token, or delete it. Changing the provider or its URL requires a new token. Deleting a profile removes it and its token from Onyxia; the token still exists on the forge, revoke it there if you no longer need it.

{% hint style="info" %}
If you had saved a GitHub personal access token in a previous version of Onyxia, it has been moved to a `github.com` profile.
{% endhint %}

## Clone a repository when launching a service

In the launcher of a compatible service, the Git repository field offers two modes:

* **My repositories**: choose a Git profile, then search your repositories. Recently active ones are suggested. Select a branch, or leave it empty for the default branch.
* **Git URL**: paste the HTTPS clone URL of any repository. If it isn't on a connected provider, it is cloned without authentication: the repository must be public.

{% hint style="warning" %}
Never put a username or token in the URL: it would be saved with the launch configuration and in the shared links. Connect the provider instead.
{% endhint %}

If a provider isn't connected yet, you can sign in or add a token directly from the launcher.

When you select **Launch**, Onyxia may create a token dedicated to the service or ask you to authorize it in a popup, depending on your organization's configuration. If the popup is blocked or the authorization fails, nothing is launched: allow popups and launch again.

## In the service

For every connected provider, the service is configured with:

* your commit identity on that forge (`user.name` and `user.email`), used for the repositories hosted there;
* credentials for `git clone`, `git pull` and `git push` over HTTPS, without typing anything.

Some providers rely on a device code: on the first `git` command towards that forge, a terminal of the service shows a link and a code to enter in your browser.

Changes in **My account > Git** apply to the services you launch afterwards; they don't update an already running service.

## Where credentials are stored

Your personal access tokens and custom profiles are stored with your Onyxia account settings:

* With Vault, they are saved in your Vault-backed user configuration.
* Without Vault, they are saved in the browser's local storage and are available only in that browser profile.

Your sign-in sessions with the forges are kept in the browser. Credentials created for a service are only sent to that service: they are never saved in your launch configurations.
