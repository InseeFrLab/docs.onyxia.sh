---
description: Onyxia's JSON Schema extention
icon: plus
---

# x-onyxia

Onyxia defines a custom extension to the [JSON Schema spec](json-schema-support.md). It adds Onyxia-specific properties under a reserved key: `x-onyxia`.

The main use case is per-user defaults based on identity. For example, you can inject the right Git and S3 credentials for each user.

### overwriteDefaultWith

Let's consider a sample of the `values.schema.json` of the InseeFrLab/helm-charts-interactive-services' Jupyter chart:

<pre class="language-json" data-title="values.schema.json"><code class="lang-json">"git": {
    "description": "Git user configuration",
    "type": "object",
    "properties": {
        "enabled": {
            "type": "boolean",
            "description": "Add git config inside your environment",
            "default": true
        },
        "name": {
            "type": "string",
            "description": "user name for git",
            "default": "",
<strong>            "x-onyxia": {
</strong><strong>                "overwriteDefaultWith": "{{git.name}}"
</strong><strong>            },
</strong>            "hidden": {
                "value": false,
                "path": "git/enabled"
            }
        },
        "email": {
            "type": "string",
            "description": "user email for git",
            "default": "",
<strong>            "x-onyxia": {
</strong><strong>                "overwriteDefaultWith": "{{git.email}}"
</strong><strong>            },
</strong>            "hidden": {
                "value": false,
                "path": "git/enabled"
            }
        },
        "cache": {
            "type": "string",
            "description": "duration in seconds of the credentials cache duration",
            "default": "",
<strong>            "x-onyxia": {
</strong><strong>                "overwriteDefaultWith": "{{git.credentials_cache_duration}}"
</strong><strong>            },
</strong>            "hidden": {
                "value": false,
                "path": "git/enabled"
            }
        },
        "token": {
            "type": "string",
            "description": "personal access token",
            "default": "",
<strong>            "x-onyxia": {
</strong><strong>                "overwriteDefaultWith": "{{git.token}}"
</strong><strong>            },
</strong>            "hidden": {
                "value": false,
                "path": "git/enabled"
            }
        },
        "repository": {
            "type": "string",
            "description": "Repository url",
            "default": "",
            "hidden": {
                "value": false,
                "path": "git/enabled"
            }
        },
        "branch": {
            "type": "string",
            "description": "Brach automatically checkout",
            "default": "",
            "hidden": {
                "value": "",
                "path": "git/repository"
            }
        }
    }
},
</code></pre>

And it translates into this:

{% embed url="https://user-images.githubusercontent.com/6702424/177571819-f2e1b4ef-ecd1-479b-a5a1-658d87d7c7c0.png" %}

Note the `"git.name"`, `"git.email"` and `"git.token"`, this enables [onyxia-web](https://github.com/InseeFrLab/onyxia-web) to pre fill the fields.

If the user took the time to fill its profile information, [onyxia-web](https://github.com/InseeFrLab/onyxia-web) knows what is the Git **username**, **email** and **personal access token** of the user.

![The onyxia user profile](<../../../.gitbook/assets/image (24).png>)

[Here](https://github.com/InseeFrLab/onyxia/blob/main/web/src/core/ports/OnyxiaApi/XOnyxia.ts) is defined the structure of the context that you can use in the `overwriteDefaultWith` field:

```typescript

export type XOnyxiaParams = {
    /**
     * This is where you can reference values from the onyxia context so that they
     * are dynamically injected by the Onyxia launcher.
     *
     * Examples:
     * "overwriteDefaultWith": "user.email" ( You can also write "{{user.email}}" it's equivalent )
     * "overwriteDefaultWith": "{{project.id}}-{{k8s.randomSubdomain}}.{{k8s.domain}}"
     * "overwriteDefaultWith": [ "a hardcoded value", "some other hardcoded value", "{{region.oauth2.clientId}}" ]
     * "overwriteDefaultWith": { "foo": "bar", "bar": "{{region.oauth2.clientId}}" }
     */
    overwriteDefaultWith?: string | Stringifyable[] | Record<string, Stringifyable>;
    overwriteListEnumWith?: string | Stringifyable[];
    hidden?: boolean;
    readonly?: boolean;
};

export type XOnyxiaContext = {
    user: {
        idep: string;
        name: string;
        email: string;
        password: string;
        ip: string;
        darkMode: boolean;
        lang: "en" | "fr" | "zh-CN" | "no" | "fi" | "nl" | "it" | "es" | "de";
        /**
         * Decoded JWT OIDC ID token of the user launching the service.
         *
         * Sample value:
         * {
         *   "sub": "9000ffa3-5fb8-45b5-88e4-e2e869ba3cfa",
         *   "name": "Joseph Garrone",
         *   "aud": ["onyxia", "minio-datanode"],
         *   "groups": [
         *       "USER_ONYXIA",
         *       "codegouv",
         *       "onyxia",
         *       "sspcloud-admin",
         *   ],
         *   "preferred_username": "jgarrone",
         *   "given_name": "Joseph",
         *   "locale": "en",
         *   "family_name": "Garrone",
         *   "email": "joseph.garrone@insee.fr",
         *   "policy": "stsonly",
         *   "typ": "ID",
         *   "azp": "onyxia",
         *   "email_verified": true,
         *   "realm_access": {
         *       "roles": ["offline_access", "uma_authorization", "default-roles-sspcloud"]
         *   }
         * }
         */
        decodedIdToken: Record<string, unknown>;
        accessToken: string;
        refreshToken: string;
        // See: https://docs.onyxia.sh/v/v10/admin-doc/catalog-of-services/customize-your-charts/declarative-user-profile
        profile: Record<string, Stringifyable> | undefined;
    };
    service: {
        oneTimePassword: string;
    };
    project: {
        id: string;
        password: string;
        basic: string;
    };
    git: {
        name: string;
        email: string;
        credentials_cache_duration: number;
        token: string | undefined;
    };
    vault:
        | {
              VAULT_ADDR: string;
              VAULT_TOKEN: string | undefined;
              VAULT_MOUNT: string;
              VAULT_TOP_DIR: string;
          }
        | undefined;
    s3:
        | {
              profileName: string;
              AWS_ACCESS_KEY_ID: string | undefined;
              AWS_SECRET_ACCESS_KEY: string | undefined;
              AWS_SESSION_TOKEN: string | undefined;
              AWS_DEFAULT_REGION: string;
              AWS_S3_ENDPOINT: string;
              port: number;
              pathStyleAccess: boolean;
              /**
               * If true the bucket's (directory) should be accessible without any credentials.
               * In this case s3.AWS_ACCESS_KEY_ID, s3.AWS_SECRET_ACCESS_KEY and s3.AWS_SESSION_TOKEN
               * are undefined.
               */
              isAnonymous: boolean;
          }
        | undefined;
    s3_array: {
        profileName: string;
        AWS_ACCESS_KEY_ID: string | undefined;
        AWS_SECRET_ACCESS_KEY: string | undefined;
        AWS_SESSION_TOKEN: string | undefined;
        AWS_DEFAULT_REGION: string;
        AWS_S3_ENDPOINT: string;
        port: number;
        pathStyleAccess: boolean;
        /**
         * If true the bucket's (directory) should be accessible without any credentials.
         * In this case s3.AWS_ACCESS_KEY_ID, s3.AWS_SECRET_ACCESS_KEY and s3.AWS_SESSION_TOKEN
         * are undefined.
         */
        isAnonymous: boolean;
    }[];
    region: {
        defaultIpProtection: boolean | undefined;
        defaultNetworkPolicy: boolean | undefined;
        allowedURIPattern: string;
        customValues: Record<string, unknown> | undefined;
        kafka:
            | {
                  url: string;
                  topicName: string;
              }
            | undefined;
        tolerations: unknown[] | undefined;
        from: unknown[] | undefined;
        nodeSelector: Record<string, unknown> | undefined;
        startupProbe: Record<string, unknown> | undefined;
        sliders: Record<
            string,
            {
                sliderMin: number;
                sliderMax: number;
                sliderStep: number;
                sliderUnit: string;
            }
        >;
        resources:
            | {
                  cpuRequest?: `${number}${string}`;
                  cpuLimit?: `${number}${string}`;
                  memoryRequest?: `${number}${string}`;
                  memoryLimit?: `${number}${string}`;
                  disk?: `${number}${string}`;
                  gpu?: `${number}`;
              }
            | undefined;
        openshiftSCC:
            | {
                  scc: string;
                  enabled: boolean;
              }
            | undefined;
    };
    k8s: {
        domain: string;
        ingressClassName: string | undefined;
        ingress: boolean | undefined;
        route: boolean | undefined;
        istio:
            | {
                  enabled: boolean;
                  gateways: string[];
              }
            | undefined;
        randomSubdomain: string;
        initScriptUrl: string;
        useCertManager: boolean;
        certManagerClusterIssuer: string | undefined;
    };
    proxyInjection:
        | {
              enabled: boolean | undefined;
              httpProxyUrl: string | undefined;
              httpsProxyUrl: string | undefined;
              noProxy: string | undefined;
          }
        | undefined;
    packageRepositoryInjection:
        | {
              cranProxyUrl: string | undefined;
              condaProxyUrl: string | undefined;
              packageManagerUrl: string | undefined;
              pypiProxyUrl: string | undefined;
          }
        | undefined;
    certificateAuthorityInjection:
        | {
              cacerts: string | undefined;
              pathToCaBundle: string | undefined;
          }
        | undefined;
};

assert<Equals<XOnyxiaContext["user"]["lang"], Language>>();

```

You can also concatenate string values using by wrapping the XOnyxia targeted values in `{{}}`.

{% code title="values.shema.json" %}
```json
"hostname": {
  "type": "string",
  "form": true,
  "title": "Hostname",
  "x-onyxia": {
    "overwriteDefaultWith": "{{project.id}}-{{k8s.randomSubdomain}}.{{k8s.domain}}"
  }
}
```
{% endcode %}

### overwriteListEnumWith

This is an option for customizing the options of the forms fields rendered as select.

<figure><img src="../../../.gitbook/assets/image (3).png" alt="" width="375"><figcaption><p>Example of select form field in the onyxia launcher</p></figcaption></figure>

In your values shema such a field would be defined like:

{% code title="values.shema.json" %}
```json
"pullPolicy": {
    "type": "string",
    "default": "IfNotPresent",
    "listEnum": [
        "IfNotPresent",
        "Always",
        "Never"
    ]
}
```
{% endcode %}

But what if you want to dynamically generate the option? For this you can use the overwriteListEnumWith x-onyxia option.\
For example if you need to let the user select one of the groups he belongs to you can write:

<pre class="language-json" data-title="values.schema.json"><code class="lang-json">"group": {
  "type": "string",
<strong>  "default": "",
</strong><strong>  "listEnum": [""],
</strong>  "x-onyxia": {
<strong>    "overwriteDefaultWith": "{{user.decodedIdToken.groups[0]}}",
</strong><strong>    "overwriteListEnumWith": "{{user.decodedIdToken.groups}}"
</strong>  }
}
</code></pre>

### overwriteSchemaWith

See: [override-schema-for-a-specific-instance.md](../override-schema-for-a-specific-instance.md "mention")
