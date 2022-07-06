---
description: Contributing to the helm charts
---

# 🔬 Catalog of services



Every Onyxia instance may or may not have it's own catalog. &#x20;

If you take for example [this instance](https://datalab.sspcloud.fr/catalog) it offers two catalogs, [InseeFrLab datascience](https://github.com/InseeFrLab/helm-charts-datascience) and [InseeFrLab R Shiny apps](https://github.com/InseeFrLab/helm-charts-shiny-apps). &#x20;

You can always find the source of the catalog by clicking on the "contribute to the... " link.

![https://datalab.sspcloud.fr/catalog](<../.gitbook/assets/image (5).png>)

If you take [this other instance](https://sill-demo.etalab.gouv.fr), it has only one catalog, [helm-charts-sill](https://github.com/etalab/helm-charts-sill).

![https://sill-demo.etalab.gouv.fr/catalog](<../.gitbook/assets/image (9).png>)







In order to contribute you have to be familiar with [Helm](https://helm.sh/) and to be familiar with Helm you need to be familiar with [Kubernetes objects](https://kubernetes.io/docs/concepts/overview/working-with-objects/kubernetes-objects/). &#x20;

In Onyxia, we use the `values.shema.json` to know what options should be displayed to the user at [the service configuration step](https://user-images.githubusercontent.com/6702424/177571819-f2e1b4ef-ecd1-479b-a5a1-658d87d7c7c0.png) and what default value Onyxia should inject. &#x20;

![https://helm.sh/docs/topics/charts/#the-chart-file-structure](<../.gitbook/assets/image (6).png>)

How it works in practice is that, in a Helm chart made for Onyxia the values.sheama.json isn't actually a JSON file per say but rather a [.mustache](https://mustache.github.io/) template that renders into a JSON file. &#x20;

{% embed url="https://cloud.githubusercontent.com/assets/288977/8779228/a3cf700e-2f02-11e5-869a-300312fb7a00.gif" %}

Let's consider a sample of the values.scema.json of the InseeFrLab/helm-charts-datascience's Jupyter helm chart: &#x20;

```mustache
"git": {
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
            "x-form": {
                "value": "{{git.name}}"
            },
            "hidden": {
                "value": false,
                "path": "git/enabled"
            }
        },
        "email": {
            "type": "string",
            "description": "user email for git",
            "default": "",
            "x-form": {
                "value": "{{git.email}}"
            },
            "hidden": {
                "value": false,
                "path": "git/enabled"
            }
        },
        "cache": {
            "type": "string",
            "description": "duration in seconds of the credentials cache duration",
            "default": "",
            "x-form": {
                "value": "{{git.credentials_cache_duration}}"
            },
            "hidden": {
                "value": false,
                "path": "git/enabled"
            }
        },
        "token": {
            "type": "string",
            "description": "personal access token",
            "default": "",
            "x-form": {
                "value": "{{git.token}}"
            },
            "hidden": {
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
```

And it translate into this:

{% embed url="https://user-images.githubusercontent.com/6702424/177571819-f2e1b4ef-ecd1-479b-a5a1-658d87d7c7c0.png" %}

Note the \{{git.name\}} \{{git.email\}} and \{{git.token\}} in the the values.shema.json, this enables onyxia-web to pre fill the fields. &#x20;

If the user took the time to fill it's profile information, onyxia-web know what is the Git username, email and personal access token of the user. &#x20;

![](<../.gitbook/assets/image (7).png>)

[Here are the values](https://github.com/InseeFrLab/onyxia-web/blob/ea2580954d50b1acedc03867353b6c26ab27eb79/src/core/ports/OnyxiaApiClient.ts#L139-L190) that you can use in the `values.shema.json` mustache template:&#x20;

```typescript
export type MustacheParams = {
    user: {
        idep: string;
        name: string;
        email: string;
        password: string;
        ip: string;
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
        token: string | null;
    };
    vault: {
        VAULT_ADDR: string;
        VAULT_TOKEN: string;
        VAULT_MOUNT: string;
        VAULT_TOP_DIR: string;
    };
    kaggleApiToken: string | null;
    s3: {
        AWS_ACCESS_KEY_ID: string;
        AWS_SECRET_ACCESS_KEY: string;
        AWS_SESSION_TOKEN: string;
        AWS_DEFAULT_REGION: string;
        AWS_S3_ENDPOINT: string;
        AWS_BUCKET_NAME: string;
        port: number;
    };
    region: {
        defaultIpProtection: boolean | undefined;
        defaultNetworkPolicy: boolean | undefined;
        allowedURIPattern: string;
        kafka:
            | {
                  url: string;
                  topicName: string;
              }
            | undefined;
    };
    k8s: {
        domain: string;
        randomSubdomain: string;
        initScriptUrl: string;
    };
};
```

