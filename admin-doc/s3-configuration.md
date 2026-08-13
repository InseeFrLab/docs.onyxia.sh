---
icon: folder
---

# S3 Configuration

Use `onyxia.api.regions[].data.S3` to connect an Onyxia region to AWS S3 or an S3-compatible object store.

Onyxia Web uses this configuration to:

* expose administrator-defined S3 profiles in the file explorer;
* exchange the user's OIDC access token for temporary credentials through `AssumeRoleWithWebIdentity`;
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

  /** When present, this entry creates administrator-defined profiles. */
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

An S3 configuration without `sts` does not create an administrator-defined profile. Instead, it supplies the default URL, region, and path-style setting shown when a user creates a profile manually.

If `data.S3` contains several entries, Onyxia uses the first entry without `sts` for those form defaults. If every entry has `sts`, it uses the first entry.

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
* Referenced roles and policies must already exist and grant access consistent with the displayed bookmarks.
* The OIDC access token is sent to STS as the web identity token. Claim templates, however, are resolved from the corresponding decoded ID token.
* Profile names must be unique across administrator-defined and user-created profiles. Name collisions are unsupported and can cause the conflicting profiles to be discarded.
