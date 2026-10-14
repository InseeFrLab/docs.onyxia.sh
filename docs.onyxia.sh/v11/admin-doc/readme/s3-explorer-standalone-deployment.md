---
description: >-
  Deploy a standalone S3 browser with OIDC login and STS-based temporary
  credentials
icon: sign-posts-wrench
---

# S3 Explorer Standalone Deployment

Onyxia S3 Explorer is a browser-based file manager for S3-compatible object storage. Files can be previewed without leaving the explorer, including images, videos, PDFs, and text or source code with syntax highlighting.

For data workflows, CSV, JSON, and Parquet files can be explored as tabular data directly in the browser. Powered by DuckDB-Wasm, the explorer queries and streams data from object storage as needed, allowing even large files to be inspected quickly without downloading them in full first.

What makes Onyxia S3 Explorer suitable for multi-user production deployments is its native integration with your organization's identity provider and your storage provider's authorization system:

* users sign in through OpenID Connect (OIDC);
* Onyxia exchanges their OIDC access token for short-lived S3 credentials through STS, so users never need to copy or manage access keys;
* the S3 provider's roles and policies remain authoritative over which buckets and objects each user can access; and
* administrators can generate built-in bookmarks from identity claims, directing each user to their personal or project buckets.

The resulting flow is: **OIDC sign-in → STS → temporary credentials → direct browser access to S3**. Onyxia handles authentication, credential acquisition, and navigation; it does not become a data proxy or replace the authorization rules of the storage provider.

This page presents two deployment paths:

* **Quick evaluation without OIDC:** deploy the explorer with almost no configuration and browse a public bucket. This lets you try the interface before setting up identity and storage integration, but it is not intended as a production architecture.
* **Production deployment with OIDC and STS:** connect Onyxia to your identity provider and let users automatically obtain temporary, policy-scoped credentials.

If you only want to see the product running, start with the quick evaluation. If you are evaluating its production architecture, skip directly to [Production Deployment: OIDC and STS](s3-explorer-standalone-deployment.md#production-deployment-oidc-and-sts).

{% hint style="warning" %}
The explorer runs entirely in the browser: Onyxia does not proxy S3 requests. The bucket you want to browse must therefore allow the origin of the Onyxia application, for example `https://onyxia.example.com`, in its CORS configuration.
{% endhint %}

## Quick Evaluation: Deploy Without OIDC

In this mode, users create their own S3 profiles and provide an endpoint, a region, and, when required, an access key ID and secret access key. Use it to evaluate the explorer, not as a model for a multi-user production deployment.

Add the Onyxia Helm repository and create a values file:

{% code title="onyxia-values.yaml" %}
```yaml
ingress:
  enabled: true
  hosts:
    - host: onyxia.example.com

web:
  env:
    HEADER_TEXT_FOCUS: "S3 Explorer"

api:
  enabled: false
```
{% endcode %}

Install Onyxia:

```bash
helm repo add onyxia https://inseefrlab.github.io/onyxia
helm repo update

helm upgrade --install onyxia onyxia/onyxia \
  --namespace onyxia \
  --create-namespace \
  --values onyxia-values.yaml
```

Open `https://onyxia.example.com`. Onyxia prompts you to create an S3 profile. Credentials are optional for buckets that allow anonymous access.

### Try It With a Public Bucket

Create a profile with the following values:

* **Profile name:** `aws_us-west-2_anonymous`, for example
* **URL of the S3 service:** `https://s3.amazonaws.com`
* **Default region:** `us-west-2`
* **Anonymous access:** enabled

After saving the profile, navigate to `s3://multimedia-commons/` and add it to your bookmarks.

{% hint style="warning" %}
Because there is no backend in this deployment mode, user-created profiles, including any access keys, are stored in the browser's local storage. Do not enter long-lived credentials on a shared or untrusted device.
{% endhint %}

## Production Deployment: OIDC and STS

In a production multi-user deployment, Onyxia can obtain temporary credentials for each user through OIDC and STS. Your storage provider must support [`AssumeRoleWithWebIdentity`](https://docs.aws.amazon.com/STS/latest/APIReference/API_AssumeRoleWithWebIdentity.html).

{% hint style="info" %}
Onyxia does not decide which buckets a user can access. Roles and policies are configured independently in the S3/STS provider. Onyxia only authenticates the user, requests temporary credentials from STS, and displays administrator-defined bookmarks (example `s3://user-bucket-johnd/`). A bookmark does not grant access to its target.
{% endhint %}

For a complete walkthrough that deploys Kubernetes, Keycloak, MinIO, and Onyxia from scratch, follow the installation tutorial. Its **Data (S3)** section includes the standalone S3 Explorer option.

{% content-ref url="../../" %}
[..](../../)
{% endcontent-ref %}

### Example: MinIO and Keycloak

This example follows the [MinIO configuration from `onyxia-ops`](https://github.com/InseeFrLab/onyxia-ops/blob/main/apps/minio/values.yaml). It gives each user read/write access to a bucket derived from their username.

#### 1. Configure the OIDC Client and Token Claims

Create a **public OIDC client** named `onyxia-minio` in Keycloak or your OIDC provider. It must use the Authorization Code flow with PKCE and must not have a client secret.

For a user named `johnd`, tokens issued to this client must contain:

* an ID token with `preferred_username: "johnd"`; and
* a JWT access token with `preferred_username: "johnd"` and a hard-coded `policy: "stsonly"` claim.

In abbreviated form:

{% tabs %}
{% tab title="ID token" %}
Payload of the ID Token. Used to construct the bookmarks.

```json
{
  "aud": "onyxia-minio",
  "preferred_username": "johnd"
}
```
{% endtab %}

{% tab title="Access token" %}
Payload of the AccessToken sent to MinIO

```json
{
  "azp": "onyxia-minio", // Client the token was issued to.
  "aud": "minio", // Client the token is intended for.
  "preferred_username": "johnd", // Username used to template the access rules.
  "policy": "stsonly" // Required claim for the rules to apply.
}
```
{% endtab %}
{% endtabs %}

Onyxia reads `preferred_username` from the **ID token** to build the bookmark. MinIO reads `policy` and `preferred_username` from the **access token** to authorize the STS request. The username must therefore have the same value in both tokens.

#### 2. Configure MinIO's OIDC Trust and Policy

The relevant parts of the example MinIO values are:

{% code title="apps/minio/values.yaml" %}
```yaml
minio:
  oidc:
    enabled: true
    configUrl: "https://auth.example.com/realms/my-realm/.well-known/openid-configuration"
    clientId: "minio"
    clientSecret: "<MINIO_OIDC_CLIENT_SECRET>"
    claimName: "policy"
    claimPrefix: ""

  policies:
    - name: stsonly
      statements:
        - resources:
            - 'arn:aws:s3:::user-${jwt:preferred_username}'
            - 'arn:aws:s3:::user-${jwt:preferred_username}/*'
          actions:
            - "s3:*"
```
{% endcode %}

MinIO uses the access token's `policy` claim to select the `stsonly` policy. It then substitutes the token's `preferred_username` claim in the resource names. For `johnd`, the policy grants S3 operations on the `user-johnd` bucket and its objects.

The `user-` prefix is a convention in this example, not an Onyxia requirement. If your user's bucket should be named exactly `johnd`, remove the prefix from both the MinIO policy resources and the Onyxia bookmark so that they continue to match.

If the bucket does not exist, Onyxia will ask the user if they want to create it.

{% hint style="info" %}
The `minio` client shown in the MinIO values is used by the MinIO Console and has a client secret. It is separate from the public `onyxia-minio` client used by Onyxia. Never place a client secret in the Onyxia web configuration.
{% endhint %}

#### 3. Configure Onyxia

The following configuration creates a `default` profile and uses the ID token's `preferred_username` claim to create the matching personal-bucket bookmark:

{% code title="onyxia-values.yaml" %}
```yaml
ingress:
  enabled: true
  hosts:
    - host: onyxia.example.com

web:
  env:
    HEADER_TEXT_FOCUS: "S3 Explorer"
    S3: |
      {
        URL: "https://minio.example.com",
        region: "us-east-1",
        pathStyleAccess: true,
        sts: {
          role: {
            profileName: "default",
            roleARN: "",
            roleSessionName: ""
          },
          oidcConfiguration: {
            issuerURI: "https://auth.example.com/realms/my-realm",
            clientID: "onyxia-minio"
          }
        },
        bookmarks: [
          {
            s3Uri: "s3://user-$1/",
            title: { en: "Personal bucket", fr: "Bucket personnel" },
            claimName: "preferred_username",
            forProfileName: "default"
          }
        ]
      }

api:
  enabled: false
```
{% endcode %}

Install the chart with the same Helm command used in the quick-evaluation example, then open `https://onyxia.example.com`.

For `johnd`, the complete flow is:

1. Onyxia authenticates the user through the public `onyxia-minio` client.
2. Onyxia resolves the ID token's `preferred_username` and displays `s3://user-johnd/` as a bookmark.
3. Onyxia sends the access token to MinIO's STS endpoint.
4. MinIO selects the `stsonly` policy and resolves its resource to `user-johnd`.
5. MinIO returns temporary credentials that allow the browser to access that bucket.

{% hint style="info" %}
The empty `roleARN` and `roleSessionName` values are specific to MinIO's claim-based OIDC mode: MinIO derives authorization from the JWT claims instead. Other STS providers, including AWS, require valid role values. See [S3 Configuration](../s3-configuration.md) for provider-independent examples and the complete configuration reference.
{% endhint %}

## Where to Put the S3 Configuration

The location of the S3 configuration depends on whether the Onyxia API is enabled:

* **Standalone deployment:** set the configuration as JSON5 in `web.env.S3`, as shown above.
* **Full Onyxia deployment:** set it in `api.regions[].data.S3`. The Helm chart passes the first region's S3 configuration to the web application automatically.

For other identity and storage providers, see:

{% content-ref url="../s3-configuration.md" %}
[s3-configuration.md](../s3-configuration.md)
{% endcontent-ref %}

{% content-ref url="../openid-connect-configuration.md" %}
[openid-connect-configuration.md](../openid-connect-configuration.md)
{% endcontent-ref %}
