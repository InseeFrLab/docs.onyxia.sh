---
description: Enable S3 storage via MinIO S3
icon: folder
---

# Data (S3)

Onyxia exchanges the user's OIDC access token for temporary S3 credentials through [AssumeRoleWithWebIdentity](https://docs.aws.amazon.com/STS/latest/APIReference/API_AssumeRoleWithWebIdentity.html). This guide uses MinIO; other S3 systems compatible with this STS API are also supported.

We assume the [previous authentication step](user-authentication.md) configured the `minio` client and the `onyxia` token's `minio` audience, `onyxia-username`, and `minio-policy: stsonly` claims. **Onyxia reuses the token from its central `onyxia` client for MinIO STS.**

## Deploying MinIO

In Keycloak, obtain or regenerate the secret for the existing confidential `minio` client. Set it as `$KEYCLOAK_MINIO_CLIENT_SECRET` in [apps/minio/values.yaml](https://github.com/garronej/onyxia-ops/blob/main/apps/minio/values.yaml), and replace `$DOMAIN` with your domain. This secret is used by the MinIO Console; the Onyxia SPA has no client secret.

From your GitOps checkout, commit and push the configuration:

```bash
git add apps/minio/values.yaml
git commit -m "Configure MinIO authentication"
git push
```

Deploy MinIO through ArgoCD:

{% embed url="https://app.tango.us/app/embed/75b62573-7adc-4a38-b1f9-b96bb0ea50fd" %}

The supplied `stsonly` policy grants access to `user-<onyxia-username>` and its objects. The `minio-policy` claim selects this policy. For direct MinIO Console login, include `onyxia-username` on the `minio` client as described in the authentication step.

## Updating the Onyxia configuration

Merge the S3 region settings from [values-minio-enabled.yaml](https://github.com/garronej/onyxia-ops/blob/main/apps/onyxia/values-minio-enabled.yaml) into [apps/onyxia/values.yaml](https://github.com/garronej/onyxia-ops/blob/main/apps/onyxia/values.yaml), preserving your existing authentication and region settings. Replace `$DOMAIN` with your domain.

The personal bookmark must resolve to `s3://user-$1/` using the `onyxia-username` claim, matching MinIO's policy. Leave `data.S3.sts.oidcConfiguration` unset so Onyxia reuses its main access token.

```bash
git add apps/onyxia/values.yaml
git commit -m "Enable MinIO"
git push
```

For an S3 Explorer without the Onyxia API, use [values-s3-explorer-only.yaml](https://github.com/garronej/onyxia-ops/blob/main/apps/onyxia/values-s3-explorer-only.yaml) and the [standalone deployment guide](s3-explorer-standalone-deployment.md). It also uses the public `onyxia` client.

Sign in and check that you can create or access your personal bucket. Shared group buckets require additional storage policies; see [Setting up group projects](../setting-up-group-projects.md).

Next, configure Vault for secrets and saved user settings:

{% content-ref url="vault.md" %}
[Vault](vault.md)
{% endcontent-ref %}
