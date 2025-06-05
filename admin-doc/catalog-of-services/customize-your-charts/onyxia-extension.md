---
description: Onyxia has his own extension of json schema
icon: plus
---

# Onyxia extension

### \[x-onyxia] overwriteDefaultWith

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

![The onyxia user profile](<../../../.gitbook/assets/image (21).png>)

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
     *
     */
    overwriteDefaultWith?:
        | string
        | number
        | boolean
        | unknown[]
        | Record<string, unknown>;
    overwriteListEnumWith?: unknown[] | string;
    hidden?: boolean;
    readonly?: boolean;
    useRegionSliderConfig?: string;
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
    vault: {
        VAULT_ADDR: string;
        VAULT_TOKEN: string;
        VAULT_MOUNT: string;
        VAULT_TOP_DIR: string;
    };
    s3: {
        AWS_ACCESS_KEY_ID: string;
        AWS_SECRET_ACCESS_KEY: string;
        AWS_SESSION_TOKEN: string;
        AWS_DEFAULT_REGION: string;
        AWS_S3_ENDPOINT: string;
        AWS_BUCKET_NAME: string;
        port: number;
        pathStyleAccess: boolean;
        /**
         * The user is assumed to have read/write access on every
         * object starting with this prefix on the bucket
         **/
        objectNamePrefix: string;
        /**
         * Only for making it easier for charts editors.
         * <AWS_BUCKET_NAME>/<objectNamePrefix>
         * */
        workingDirectoryPath: string;
        /**
         * If true the bucket's (directory) should be accessible without any credentials.
         * In this case s3.AWS_ACCESS_KEY_ID, s3.AWS_SECRET_ACCESS_KEY and s3.AWS_SESSION_TOKEN
         * will be empty strings.
         */
        isAnonymous: boolean;
    };
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
              enabled: string | undefined;
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

This is an option for customizing the options of the forms fields rendered as select.

<figure><img src="../../../.gitbook/assets/image (1) (1).png" alt="" width="375"><figcaption><p>Example of select form field in the onyxia launcher</p></figcaption></figure>

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

### \[x-onyxia] overwriteSchemaWith

Certain elements of a Helm chart should be customized for each instance of Onyxia, such as resource requests and limits, node selectors and tolerations. For this purpose, chart developers can use `x-onyxia.overwriteSchemaWith` to allow administrators to override specific parts of the schema. Our default charts use this specification.

{% code title="values.shema.json" %}
```json
"nodeSelector": {
    "type": "object",
    "description": "NodeSelector",
    "default": {},
    "x-onyxia": {
        "overwriteSchemaWith": "nodeSelector.json"
    }
}
```
{% endcode %}

You can see [here](https://github.com/InseeFrLab/onyxia-api/tree/main/onyxia-api/src/main/resources/schemas) the list of default schemas included in the Onyxia API. We also provide examples demonstrating how you [can customize your services using our interactive services charts with the provided schemas](https://github.com/InseeFrLab/helm-charts-interactive-services/).

The following node selector schema provided by Onyxia API is a generic definition, which may not provide the best experience for a specific Kubernetes cluster in Onyxia.

{% code title="nodeSelector.json" %}
```json
{
    "$schema": "http://json-schema.org/draft-07/schema#",
    "title": "Node Selector",
    "type": "object",
    "description": "Node selector constraints for the pod",
    "additionalProperties": {
      "type": "string",
      "description": "Key-value pairs to select nodes"
    }
}
```
{% endcode %}

As an administrator of Onyxia, you can provide your own schemas to refine and restrict the initial schemas provided in the Helm chart.

#### node selectors

You can provide this schema to allow your users to choose between SSD or HDD disk types, and A2 or H100 NVIDIA GPUs. Any other values or labels are disallowed, and Onyxia will reject starting a service that does not comply with the provided schema.

{% code title="nodeSelector.json" %}
```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "Node Selector",
  "type": "object",
  "properties": {
    "disktype": {
      "description": "The type of disk",
      "type": "string",
      "enum": ["ssd", "hdd"],
      "default": "ssd"
    },
    "gpu": {
      "description": "The type of GPU",
      "type": "string",
      "enum": ["A2", "H100"],
      "default": "A2"
    }
  },
  "additionalProperties": false //any other label is disallowed
}
```
{% endcode %}

#### rolebindings for IDE pods

This is the default role for IDE pods in our charts. It is very permissive, and you may want to restrict it to view-only access.

{% code title="ide/role.json" %}
```json
{
    "$schema": "http://json-schema.org/draft-07/schema#",
    "title": "Role",
    "type": "object",
    "properties": {
        "enabled": {
            "type": "boolean",
            "description": "allow your service to access your namespace ressources",
            "default": true
        },
        "role": {
            "type": "string",
            "description": "bind your service account to this kubernetes default role",
            "default": "view",
            "enum": [
                "view",
                "edit",
                "admin"
            ]
        }
    }
}

```
{% endcode %}

Here is the refined version

{% code title="ide/role.json" %}
```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "Role",
  "type": "object",
  "properties": {
    "enabled": {
      "type": "boolean",
      "const": true,
      "description": "This value must always be true, allowing your service to access your namespace resources."
    },
    "role": {
      "type": "string",
      "const": "view",
      "description": "This value must always be 'view', binding your service account to this Kubernetes default role.",
    }
  }
}

```
{% endcode %}

#### resources for IDE

You may want to modify the slide bar for resources

{% code title="ide/resources.json" %}
```json
{
    "$schema": "http://json-schema.org/draft-07/schema#",
    "title": "Resources",
    "description": "Your service will have at least the requested resources and never more than its limits. No limit for a resource and you can consume everything left on the host machine.",
    "type": "object",
    "properties": {
        "requests": {
            "description": "Guaranteed resources",
            "type": "object",
            "properties": {
                "cpu": {
                    "description": "The amount of cpu guaranteed",
                    "title": "CPU",
                    "type": "string",
                    "default": "100m",
                    "render": "slider",
                    "sliderMin": 50,
                    "sliderMax": 40000,
                    "sliderStep": 50,
                    "sliderUnit": "m",
                    "sliderExtremity": "down",
                    "sliderExtremitySemantic": "guaranteed",
                    "sliderRangeId": "cpu"
                },
                "memory": {
                    "description": "The amount of memory guaranteed",
                    "title": "memory",
                    "type": "string",
                    "default": "2Gi",
                    "render": "slider",
                    "sliderMin": 1,
                    "sliderMax": 200,
                    "sliderStep": 1,
                    "sliderUnit": "Gi",
                    "sliderExtremity": "down",
                    "sliderExtremitySemantic": "guaranteed",
                    "sliderRangeId": "memory"
                }
            }
        },
        "limits": {
            "description": "max resources",
            "type": "object",
            "properties": {
                "cpu": {
                    "description": "The maximum amount of cpu",
                    "title": "CPU",
                    "type": "string",
                    "default": "30000m",
                    "render": "slider",
                    "sliderMin": 50,
                    "sliderMax": 40000,
                    "sliderStep": 50,
                    "sliderUnit": "m",
                    "sliderExtremity": "up",
                    "sliderExtremitySemantic": "Maximum",
                    "sliderRangeId": "cpu"
                },
                "memory": {
                    "description": "The maximum amount of memory",
                    "title": "Memory",
                    "type": "string",
                    "default": "50Gi",
                    "render": "slider",
                    "sliderMin": 1,
                    "sliderMax": 200,
                    "sliderStep": 1,
                    "sliderUnit": "Gi",
                    "sliderExtremity": "up",
                    "sliderExtremitySemantic": "Maximum",
                    "sliderRangeId": "memory"
                }
            }
        }
    }
}

```
{% endcode %}

<pre class="language-json" data-title="ide/resources.json"><code class="lang-json"><strong>{
</strong>    "$schema": "http://json-schema.org/draft-07/schema#",
    "title": "Resources",
    "description": "Your service will have at least the requested resources and never more than its limits. No limit for a resource and you can consume everything left on the host machine.",
    "type": "object",
    "properties": {
        "requests": {
            "description": "Guaranteed resources",
            "type": "object",
            "properties": {
                "cpu": {
                    "description": "The amount of cpu guaranteed",
                    "title": "CPU",
                    "type": "string",
                    "default": "100m",
                    "render": "slider",
                    "sliderMin": 50,
                    "sliderMax": 10000,
                    "sliderStep": 50,
                    "sliderUnit": "m",
                    "sliderExtremity": "down",
                    "sliderExtremitySemantic": "guaranteed",
                    "sliderRangeId": "cpu"
                },
                "memory": {
                    "description": "The amount of memory guaranteed",
                    "title": "memory",
                    "type": "string",
                    "default": "2Gi",
                    "render": "slider",
                    "sliderMin": 1,
                    "sliderMax": 200,
                    "sliderStep": 1,
                    "sliderUnit": "Gi",
                    "sliderExtremity": "down",
                    "sliderExtremitySemantic": "guaranteed",
                    "sliderRangeId": "memory"
                }
            }
        },
        "limits": {
            "description": "max resources",
            "type": "object",
            "properties": {
                "cpu": {
                    "description": "The maximum amount of cpu",
                    "title": "CPU",
                    "type": "string",
                    "default": "5000m",
                    "render": "slider",
                    "sliderMin": 50,
                    "sliderMax": 10000,
                    "sliderStep": 50,
                    "sliderUnit": "m",
                    "sliderExtremity": "up",
                    "sliderExtremitySemantic": "Maximum",
                    "sliderRangeId": "cpu"
                },
                "memory": {
                    "description": "The maximum amount of memory",
                    "title": "Memory",
                    "type": "string",
                    "default": "50Gi",
                    "render": "slider",
                    "sliderMin": 1,
                    "sliderMax": 200,
                    "sliderStep": 1,
                    "sliderUnit": "Gi",
                    "sliderExtremity": "up",
                    "sliderExtremitySemantic": "Maximum",
                    "sliderRangeId": "memory"
                }
            }
        }
    }
}
</code></pre>
