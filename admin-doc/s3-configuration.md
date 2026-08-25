---
icon: folder
---

# S3 Configuration

Use `onyxia.api.regions[].data.S3` to connect an Onyxia region to AWS S3 or an S3-compatible object store.

Onyxia Web uses this configuration to:

* expose administrator-defined S3 profiles in the file explorer;
* exchange the user's OIDC access token for temporary credentials through `AssumeRoleWithWebIdentity`;
* expose public S3 data through an anonymous profile and generate links to public folders;
* inject the selected profile and its credentials into services such as Jupyter and RStudio.

The browser talks directly to the S3 and STS endpoints. Onyxia API does not proxy these requests, create IAM roles, or install bucket policies. Configure the S3 provider, OIDC trust, roles, and policies separately, and allow requests from the Onyxia Web origin in the S3 provider's CORS configuration.

The [installation guide](readme/data-s3.md) demonstrates a basic [MinIO](https://min.io/) deployment. This page documents the region configuration consumed by Onyxia Web.

## Minimal MinIO Example

This example defines a single administrator-managed profile named `default`. The bookmark resolves to a bucket named after the user's `preferred_username` claim.

{% code title="apps/onyxia/values.yaml" %}
```yaml
onyxia:
  api:
    regions:
      - {
          "id": "default",
          "data": {
            "S3": {
              "URL": "https://minio.lab.example.com",
              "sts": {
                "role": {
                  "profileName": "default",
                  "roleARN": "",
                  "roleSessionName": ""
                },
                "oidcConfiguration": {
                  "clientID": "onyxia-minio"
                }
              },
              "bookmarks": [
                {
                  "s3Uri": "s3://$1/",
                  "title": "Personal Bucket",
                  "claimName": "preferred_username",
                  "forProfileName": "default"
                }
              ]
            }
          }
        }
```
{% endcode %}

`roleARN` and `roleSessionName` must be present in the Onyxia configuration, but they can exceptionally be empty for MinIO. Onyxia omits empty values from the STS request. In MinIO's claim-based OIDC mode, when `RoleArn` is absent, MinIO determines the user's authorization from the configured policy claim in the JWT. MinIO's `AssumeRoleWithWebIdentity` endpoint also does not require a role session name.

To configure MinIO so that users automatically receive temporary credentials that gives them read/write access to a bucket that matches their username (the preferred\_username claim in the ID and Access token) see [this example](https://github.com/InseeFrLab/paris-sspcloud/blob/master/apps/onyxia-aws/values.yaml).

This is MinIO-specific. Providers such as AWS STS require a valid role ARN and role session name.

For a user whose S3 OIDC ID token contains:

```json
{
  "preferred_username": "alice"
}
```

Onyxia exposes the following profile:

```json
{
  "profileName": "default",
  "bookmarks": ["s3://alice/"]
}
```

If the `alice` bucket does not exist, the file explorer can offer to create it. Whether creation succeeds depends on the permissions granted by MinIO.

See [OIDC Configuration for Services Onyxia Connects To](openid-connect-configuration.md#oidc-configuration-for-services-onyxia-connects-to) for the complete `oidcConfiguration` format. When this object is omitted or only partially specified, Onyxia reuses the corresponding values from its main OIDC configuration.

## Multiple Profiles From Claims

`data.S3` accepts either one S3 configuration object or an array of them. Within one S3 configuration, `sts.role` also accepts either one role or an array.

A role with a `claimName` can produce several profiles. If the claim is an array of strings, Onyxia resolves the role once for every accepted value.

The following example creates:

* one personal profile named `default`;
* one `project-*` profile per group, excluding groups whose names start with `USER_ONYXIA`;
* a public bookmark attached to the personal and project profiles.

{% code title="apps/onyxia/values.yaml" %}
```yaml
onyxia:
  api:
    regions:
      - {
          "id": "default",
          "data": {
            "S3": {
              "URL": "https://ceph.lab.sspcloud.fr",
              "sts": {
                "role": [
                  {
                    "profileName": "default",
                    "roleARN": "arn:aws:iam::123456789012:role/$1",
                    "roleSessionName": "onyxia-personal-bucket",
                    "claimName": "preferred_username"
                  },
                  {
                    "profileName": "project-$1",
                    "roleARN": "arn:aws:iam::329456783432:role/projet-$1",
                    "roleSessionName": "onyxia-project-bucket-$1",
                    "claimName": "groups",
                    "excludedClaimPattern": "^USER_ONYXIA.*"
                  }
                ],
                "oidcConfiguration": {
                  "clientID": "onyxia-ceph"
                }
              },
              "bookmarks": [
                {
                  "s3Uri": "s3://$1/",
                  "title": "Personal Bucket",
                  "claimName": "preferred_username",
                  "forProfileName": "default"
                },
                {
                  "s3Uri": "s3://project-$1/",
                  "title": "$1 Reserved Bucket",
                  "claimName": "groups",
                  "excludedClaimPattern": "^USER_ONYXIA.*",
                  "forProfileName": "project-$1"
                },
                {
                  "s3Uri": "s3://donnees-insee/diffusion/",
                  "title": {
                    "fr": "Données de diffusion",
                    "en": "Dissemination Data"
                  },
                  "forProfileName": ["default", "project-*"]
                }
              ]
            }
          }
        }
```
{% endcode %}

For this S3 OIDC ID token:

```json
{
  "preferred_username": "johnd",
  "groups": ["sspcloud", "codegouv", "USER_ONYXIA_admin"]
}
```

Onyxia resolves:

```json
[
  {
    "profileName": "default",
    "bookmarks": [
      "s3://johnd/",
      "s3://donnees-insee/diffusion/"
    ]
  },
  {
    "profileName": "project-sspcloud",
    "bookmarks": [
      "s3://project-sspcloud/",
      "s3://donnees-insee/diffusion/"
    ]
  },
  {
    "profileName": "project-codegouv",
    "bookmarks": [
      "s3://project-codegouv/",
      "s3://donnees-insee/diffusion/"
    ]
  }
]
```

No profile is generated for `USER_ONYXIA_admin` because it matches `excludedClaimPattern`.

## Anonymous Profiles and Public-Folder Sharing

Set `anonymousProfileName` to create an administrator-defined S3 profile that does not use STS or static credentials. Requests made through this profile are unsigned and can access only the buckets, prefixes, and objects that the S3 provider allows anonymous users to access.

An anonymous profile is also required to enable sharing links for public folders in the S3 explorer. For an authenticated profile, configure an anonymous profile with the same `URL` and `region`. The simplest approach is to add `anonymousProfileName` to the same S3 configuration:

{% code title="apps/onyxia/values.yaml" %}
```yaml
onyxia:
  api:
    regions:
      - {
          "id": "default",
          "data": {
            "S3": {
              "URL": "https://s3.lab.example.com",
              "region": "us-east-1",
              "pathStyleAccess": true,
              "sts": {
                "role": {
                  "profileName": "default",
                  "roleARN": "arn:aws:iam::123456789012:role/onyxia-$1",
                  "roleSessionName": "onyxia-$1",
                  "claimName": "preferred_username"
                }
              },
              "anonymousProfileName": "public"
            }
          }
        }
```
{% endcode %}

This configuration creates two profiles for the same S3 endpoint:

* `default`, which obtains temporary credentials through STS;
* `public`, which sends no credentials.

When a folder is public, Onyxia can generate a sharing link that opens it with the `public` profile. The recipient can follow that link without signing in to Onyxia.

{% hint style="warning" %}
`anonymousProfileName` does not make any S3 data public. The folder must already be covered by a public bucket policy, or a user with sufficient permissions must make it public through the S3 explorer. The S3 provider must permit the required anonymous list and read operations and allow the Onyxia Web origin in its CORS configuration.
{% endhint %}

### Anonymous-Only S3 Profile

An S3 configuration can omit `sts` and define only `anonymousProfileName`. This creates a credential-free profile without creating any STS-backed profile:

{% code title="apps/onyxia/values.yaml" %}
```yaml
onyxia:
  api:
    regions:
      - {
          "id": "default",
          "data": {
            "S3": {
              "URL": "https://s3.public.example.com",
              "region": "us-east-1",
              "pathStyleAccess": true,
              "anonymousProfileName": "public",
              "bookmarks": [
                {
                  "s3Uri": "s3://open-data/",
                  "title": "Open Data",
                  "forProfileName": "public"
                }
              ]
            }
          }
        }
```
{% endcode %}

Here, Onyxia exposes the `public` profile and accesses `s3://open-data/` with unsigned requests. Access succeeds only if the S3 provider's policies allow it.

## Configuration Reference

The following type describes the complete `data.S3` configuration accepted by Onyxia Web:

```typescript
type RegionData = {
  S3?: S3Config | S3Config[];
};

type S3Config = {
  /** S3 API endpoint. */
  URL: string;

  /**
   * Region passed to both the S3 and STS clients.
   * The clients use "us-east-1" when this is omitted.
   */
  region?: string;

  /**
   * true:  https://s3.example.com/bucket/key
   * false: https://bucket.s3.example.com/key
   * Default: true
   */
  pathStyleAccess?: boolean;

  /** When present, each resolved role creates an STS-backed profile. */
  sts?: {
    /** STS endpoint. Defaults to S3Config.URL. */
    URL?: string;

    /**
     * Requested temporary-credential lifetime in seconds.
     * Default requested by Onyxia: 604800 (seven days).
     */
    durationSeconds?: number;

    /** Each resolved role creates one profile. */
    role: StsRole | StsRole[];

    /** Partial OIDC override for this S3 service. */
    oidcConfiguration?: OidcConfiguration;
  };

  /**
   * Creates an administrator-defined profile that sends no credentials.
   * An anonymous profile also enables sharing links for public folders on
   * the same S3 endpoint and region.
   */
  anonymousProfileName?: string;

  /** Read-only, administrator-defined bookmarks in the S3 explorer. */
  bookmarks?: Bookmark[];
};

type StsRole = {
  profileName: string;
  roleARN: string;
  roleSessionName: string;

  /** When set, resolve this role from the named ID-token claim. */
  claimName?: string;
  includedClaimPattern?: string;
  excludedClaimPattern?: string;
};

type Bookmark = {
  s3Uri: string;
  title: LocalizedString;

  /**
   * Attach the bookmark only to these profiles.
   * Supports a string, an array, and * wildcards.
   * When omitted, attach it to every profile from this S3Config.
   */
  forProfileName?: string | string[];

  /** When set, resolve this bookmark from the named ID-token claim. */
  claimName?: string;
  includedClaimPattern?: string;
  excludedClaimPattern?: string;
};

type LocalizedString = string | Record<string, string>;

type OidcConfiguration = {
  issuerURI?: string;
  clientID?: string;
  extraQueryParams?: string;
  scope?: string;
  idleSessionLifetimeInSeconds?: number | string;
};
```

The configured `durationSeconds` is only a request. The STS provider can reject it or limit the resulting credential lifetime. In particular, Onyxia's seven-day default may be too high for some providers, so set an explicit value compatible with your STS service.

### Claim Expansion and Templates

`claimName` is read from the decoded ID token produced by the OIDC configuration used for S3. Dot notation is supported for nested claims, for example `realm_access.roles`.

The claim must be a string or an array of strings:

* a string resolves one role or bookmark;
* an array resolves one role or bookmark per accepted value;
* a missing claim resolves nothing for that entry.

The claim filters are JavaScript regular expressions. Resolution works as follows:

1. `excludedClaimPattern` is tested first. A matching value is discarded.
2. `includedClaimPattern` is then applied. If omitted, it defaults to `^(.+)$`.
3. `$1`, `$2`, and subsequent placeholders are replaced with capture groups from the included match.

Role templates are supported in:

* `roleARN`;
* `roleSessionName`;
* `profileName`.

Bookmark templates are supported in:

* `s3Uri`;
* `title`, including every localized value;
* `forProfileName`.

Without `claimName`, Onyxia creates exactly one role or bookmark and treats `$1` literally.

### Bookmark Profile Selection

`forProfileName` controls which resolved profiles receive a bookmark:

* omit it to attach the bookmark to every profile generated from the same S3 configuration;
* use a string for one selector;
* use an array for several selectors;
* use `*` within a selector as a wildcard, for example `project-*`.

The selector filters bookmarks in the UI. It does not grant S3 permissions.

### Defaults for User-Created Profiles

An S3 configuration with neither `sts` nor `anonymousProfileName` does not create an administrator-defined profile. Instead, it supplies the default URL, region, and path-style setting shown when a user creates a profile manually.

If `data.S3` contains several entries, Onyxia uses the first entry with neither `sts` nor `anonymousProfileName` for those form defaults. If every entry creates an STS-backed or anonymous profile, it uses the first entry.

{% code title="apps/onyxia/values.yaml" %}
```yaml
onyxia:
  api:
    regions:
      - {
          "id": "default",
          "data": {
            "S3": [
              {
                "URL": "https://minio.lab.example.com",
                "region": "us-east-1",
                "pathStyleAccess": true
              },
              {
                "URL": "https://ceph.lab.example.com",
                "region": "us-east-1",
                "pathStyleAccess": true,
                "sts": {
                  "role": {
                    "profileName": "default",
                    "roleARN": "arn:aws:iam::123456789012:role/onyxia-$1",
                    "roleSessionName": "onyxia-$1",
                    "claimName": "preferred_username"
                  },
                  "oidcConfiguration": {
                    "clientID": "onyxia-ceph"
                  }
                }
              }
            ]
          }
        }
```
{% endcode %}

Here, the MinIO entry only supplies defaults for the manual profile form. The Ceph entry creates the administrator-defined `default` profile.

## Operational Requirements

* The S3 and STS endpoints must be reachable from users' browsers.
* The S3 provider must allow the Onyxia Web origin through CORS.
* The STS provider must trust the issuer and client configured in `sts.oidcConfiguration`.
* Anonymous profiles and public-folder sharing require the S3 provider to permit unsigned list and read requests for the relevant buckets and prefixes.
* Referenced roles and policies must already exist and grant access consistent with the displayed bookmarks.
* The OIDC access token is sent to STS as the web identity token. Claim templates, however, are resolved from the corresponding decoded ID token.
* Profile names must be unique across administrator-defined and user-created profiles. Name collisions are unsupported and can cause the conflicting profiles to be discarded.
