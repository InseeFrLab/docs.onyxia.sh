---
icon: folder
---

# S3 Configuration

Configuration parameters for integrating your Onyxia instance with S3 or an S3-compatible object store.

Onyxia uses S3 in three ways:

* It provides a file explorer backed by your S3 server.
* It requests temporary S3 credentials for users through STS and OpenID Connect.
* It injects those credentials into services such as Jupyter or RStudio.

[The installation guide](readme/data-s3.md) shows a basic [MinIO](https://min.io/) setup. This page covers the full configuration model for any S3-compatible provider.

Onyxia talks to S3 directly from the user's browser. It does not proxy S3 calls through the backend API.

The S3 configuration in Onyxia describes the access model exposed in the UI. It does not create roles or policies on your S3 server.

You must configure your S3 provider and its OIDC integration separately so that:

* the STS provider trusts your OIDC issuer;
* the requested roles already exist;
* users actually have the permissions the UI advertises.

Onyxia exchanges the user's OIDC access token for temporary S3 credentials with `AssumeRoleWithWebIdentity`.

Onyxia exposes S3 access through **profiles**, following the same idea as AWS CLI profiles. An administrator can define one or more profiles in `onyxia.api.regions[].data.S3`. Each profile tells Onyxia:

* which S3 endpoint to use;
* how to request temporary S3 credentials with STS;
* which profile name users will see and use in code snippets;
* which bookmarked S3 directories should appear in the file explorer.

## Minimal Example

This example creates one profile named `default` for each user. The STS role ARN and the bookmark are generated from the user's `preferred_username` claim.

{% code title="apps/onyxia/values.yaml" %}
```yaml
onyxia:
  api:
    regions:
      - id: default
        # ...
        data:
          S3:
            profileName: default
            URL: https://minio.lab.example.com
            sts:
              # See: https://docs.onyxia.sh/v/v11/admin-doc/openid-connect-configuration
              oidcConfiguration:
                clientId: onyxia-s3
            bookmarks:
              - s3Uri: "s3://$1/"
                title: Personal Bucket
                claimName: preferred_username
                forProfileName: default
```
{% endcode %}

For a user whose decoded OIDC ID token contains:

```json
{
  "preferred_username": "alice"
}
```

Onyxia exposes one S3 profile:

```yaml
profileName: default
bookmarks:
  - s3://alice/
```

If the bucket "alice" doesn't exist, Onyxia will ask the user if they want to create it.

## Multiple Profiles From Claims

`data.S3` can be a single object or an array of objects. A single S3 object can also generate several profiles when `sts.role` is an array or when a role uses a claim whose value is an array.

The following example creates:

* one personal profile named `default`;
* one project profile per user group, except groups matching `^USER_ONYXIA.*`;
* one public bookmark attached to all generated profiles.

```yaml
onyxia:
  api:
    regions:
      - id: default
        # ...
        data:
          S3:
            URL: https://ceph.lab.sspcloud.fr
            sts:
              role:
                - profileName: default
                  roleARN: arn:aws:iam::123456789012:role/$1
                  roleSessionName: onyxia-personal-bucket
                  claimName: preferred_username

                - profileName: project-$1
                  roleARN: arn:aws:iam::329456783432:role/projet-$1
                  roleSessionName: onyxia-project-bucket-$1
                  claimName: groups
                  excludedClaimPattern: "^USER_ONYXIA.*"

              oidcConfiguration:
                clientId: onyxia-ceph

            bookmarks:
              - s3Uri: "s3://$1/"
                title: Personal Bucket
                claimName: preferred_username
                forProfileName: default

              - s3Uri: "s3://project-$1/"
                title: "$1 Reserved Bucket"
                claimName: groups
                excludedClaimPattern: "^USER_ONYXIA.*"
                forProfileName: project-$1

              - s3Uri: "s3://donnees-insee/diffusion/"
                title:
                  fr: Données de diffusion
                  en: Dissemination Data
                forProfileName:
                  - default
                  - project-*
```

For a user connected with this OIDC ID token:

```json
{
  "preferred_username": "johnd",
  "groups": ["sspcloud", "codegouv", "USER_ONYXIA_admin"]
}
```

Onyxia exposes these profiles:

```yaml
- profileName: default
  bookmarks:
    - s3://johnd/
    - s3://donnees-insee/diffusion/

- profileName: project-sspcloud
  bookmarks:
    - s3://project-sspcloud/
    - s3://donnees-insee/diffusion/

- profileName: project-codegouv
  bookmarks:
    - s3://project-codegouv/
    - s3://donnees-insee/diffusion/
```

No profile is generated for `USER_ONYXIA_admin` because it matches `excludedClaimPattern`.

## Main Options

Here is an exhaustive documentation of the available options:

```typescript
type RegionData = {
  /**
   * One S3 configuration object, or an array of S3 configuration objects.
   */
  S3?: S3Config | S3Config[];
};

type S3Config = {
  /**
   * The URL of the S3 server.
   * Examples: "https://minio.lab.example.com", "https://s3.amazonaws.com".
   */
  URL: string;

  /**
   * The S3 region.
   * Optional for some S3-compatible systems such as MinIO.
   */
  region?: string;

  /**
   * Controls how object URLs are built.
   *
   * true:
   *   https://minio.lab.example.com/my-bucket/path/to/file.parquet
   *
   * false:
   *   https://my-bucket.minio.lab.example.com/path/to/file.parquet
   *
   * Default: true
   */
  pathStyleAccess?: boolean;

  /**
   * Use this only when sts.role is omitted.
   * When sts.role is present, the generated profile names come from
   * sts.role.profileName.
   * If sts.role is omitted and profileName is also omitted, Onyxia uses
   * "default" when there is only one S3 configuration.
   */
  profileName?: string;

  /**
   * STS configuration. S3 configurations with sts become administrator-defined
   * profiles. S3 configurations without sts are not profiles; they only provide
   * default endpoint values when users create their own profile manually.
   */
  sts?: {
    /**
     * STS endpoint URL.
     * For MinIO, this usually defaults to S3.URL.
     * For AWS, use "https://sts.amazonaws.com".
     */
    URL?: string;

    /**
     * Temporary credential duration, in seconds.
     * AWS maximum: 43200 seconds.
     * MinIO maximum: 604800 seconds.
     * Default requested by Onyxia: 604800 seconds.
     */
    durationSeconds?: number;

    /**
     * One role definition, or an array of role definitions.
     * Each resolved role creates one S3 profile.
     */
    role?: StsRole | StsRole[];

    /**
     * OIDC configuration used by Onyxia for the STS web identity flow.
     * See:
     * /admin-doc/openid-connect-configuration#oidc-configuration-for-services-onyxia-connects-to
     */
    oidcConfiguration?: OidcConfiguration;
  };

  /**
   * Bookmarks shown in the S3 file explorer.
   * Region-defined bookmarks are read-only from the user's point of view.
   */
  bookmarks?: Bookmarks[];
};

type StsRole = {
  roleARN: string;
  roleSessionName: string;
  profileName: string;

  /**
   * Optional claim used to generate one or more profiles.
   */
  claimName?: string;
  includedClaimPattern?: string;
  excludedClaimPattern?: string;
};

type Bookmarks = {
  s3Uri: string;

  title: LocalizedString;

  /**
   * Optional profile selector.
   * Can be a profile name, an array of profile names, or a wildcard pattern such
   * as "project-*". If omitted, the bookmark is attached to every profile
   * generated by the same S3 configuration.
   */
  forProfileName?: string | string[];

  /**
   * Optional claim used to generate one or more bookmarks.
   */
  claimName?: string;
  includedClaimPattern?: string;
  excludedClaimPattern?: string;
};
```

<details>

<summary>Templating and claim expansion rules</summary>

`claimName` refers to a claim in the decoded OIDC ID token associated with the S3 OIDC configuration. Dot notation is supported for nested claims, for example `realm_access.roles`.

The claim value must be either a string or an array of strings:

* if the value is a string, Onyxia resolves one role or bookmark;
* if the value is an array, Onyxia resolves one role or bookmark per accepted value.

`includedClaimPattern` and `excludedClaimPattern` are regular expressions:

* if `includedClaimPattern` is omitted, Onyxia behaves as if it were `^(.+)$`;
* if `excludedClaimPattern` matches a claim value, that value is ignored;
* the included pattern is then applied to the remaining values.

In templated fields, `$1`, `$2`, and so on are replaced by the capture groups of `includedClaimPattern`. With the default included pattern, `$1` is the full claim value.

Templating is available in:

* `sts.role.roleARN`;
* `sts.role.roleSessionName`;
* `sts.role.profileName`;
* `bookmarkedDirectories[].fullPath`;
* `bookmarkedDirectories[].title`;
* `bookmarkedDirectories[].description`;
* `bookmarkedDirectories[].tags`;
* `bookmarkedDirectories[].forProfileName`.

When `claimName` is not specified, the values are used literally and exactly one role or bookmark is produced.

</details>

<details>

<summary>Using one entry only for manual profile defaults</summary>

An S3 entry without `sts` does not create an administrator-defined profile. It is only used to prefill the endpoint fields when a user manually creates an S3 profile in Onyxia.

```yaml
onyxia:
  api:
    regions:
      - id: default
        # ...
        data:
          S3:
            - URL: https://minio.lab.example.com
              region: us-east-1
              pathStyleAccess: true

            - URL: https://ceph.lab.example.com
              region: us-east-1
              pathStyleAccess: true
              sts:
                role:
                  profileName: default
                  roleARN: arn:aws:iam::123456789012:role/onyxia-$1
                  roleSessionName: onyxia-$1
                  claimName: preferred_username
                oidcConfiguration:
                  clientId: onyxia-ceph
```

In this example, the first entry only provides default values for the manual creation form. The second entry creates the `default` profile.

</details>

## Operational Notes

* Onyxia does not create IAM roles or S3 bucket policies. The configured role must already grant the intended S3 permissions.
* The OIDC client configured in `sts.oidcConfiguration` must be trusted by the STS provider.
* The claims referenced by `claimName` are read from the decoded OIDC ID token.
* Prefer a profile named `default` when users should have one obvious primary S3 profile. Onyxia also uses `default` naturally in generated AWS CLI snippets.
