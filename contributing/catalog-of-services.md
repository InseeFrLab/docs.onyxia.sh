# 🔬 Catalog of services

Every Onyxia instance may or may not have it's own catalog. There is three default catalogs :

{% embed url="https://github.com/inseefrlab/helm-charts-interactive-services" %}

This collection of charts help users to launch many IDE with various binary stacks (python , R) with or without GPU support. Docker images are built [here](https://github.com/inseefrlab/images-datascience) and help us to give a homogeneous stack.

{% embed url="https://github.com/inseefrlab/helm-charts-databases" %}

This collection of charts help users to launch many databases system. Most of them are based on [bitnami/charts](https://github.com/bitnami/charts).

{% embed url="https://github.com/InseeFrLab/helm-charts-automation/" %}

This collection of charts help users to start automation tools for their datascience activity.

You can always find the source of the catalog by clicking on the "contribute to the... " link.

![https://datalab.sspcloud.fr/catalog](<../.gitbook/assets/image (5).png>)

If you take [this other instance](https://sill-demo.etalab.gouv.fr), it has only one catalog, [helm-charts-sill](https://github.com/etalab/helm-charts-sill).

![https://sill-demo.etalab.gouv.fr/catalog](<../.gitbook/assets/image (9).png>)

The available catalogs in a given Onyxia instance are configured at install time, example with datalab.sspcloud.fr:

```bash
helm repo add inseefrlab https://inseefrlab.github.io/helm-charts

DOMAIN=my-domain.net

cat << EOF > ./onyxia-values.yaml
ingress:
  enabled: true
  annotations:
    kubernetes.io/ingress.class: nginx
  hosts:
    - host: onyxia.$DOMAIN
api:
  catalogs: 
    [
       {
          "id": "ide",
          "name": "Services interactifs",
          "description": "Services for datascientists.",
          "maintainer": "innovation@insee.fr",
          "location": "https://inseefrlab.github.io/helm-charts-interactive-services",
          "status": "PROD",
          "highlightedCharts": ["jupyter-python", "rstudio", "vscode-python"],
          "type": "helm",
        },
        {
          "id": "databases",
          "name": "Bases de données",
          "description": "Services for datascientists.",
          "maintainer": "innovation@insee.fr",
          "location": "https://inseefrlab.github.io/helm-charts-databases",
          "status": "PROD",
          "highlightedCharts": ["postgresql", "elastic"],
          "type": "helm",
        },
        {
          "id": "automation",
          "name": "Automatisation",
          "description": "Services for datascientists.",
          "maintainer": "innovation@insee.fr",
          "location": "https://inseefrlab.github.io/helm-charts-automation",
          "status": "PROD",
          "highlightedCharts": ["argo-cd", "argo-workflows", "mlflow"],
          "type": "helm",
        }
    ]
  regions: 
    [
      {
        "services":{
          "expose":{
            "domain":"lab.$DOMAIN"
          }
        }
      }
    ]
EOF

helm install onyxia inseefrlab/onyxia -f onyxia-values.yaml
```

In order to contribute you have to be familiar with [Helm](https://helm.sh/) and to be familiar with Helm you need to be familiar with [Kubernetes objects](https://kubernetes.io/docs/concepts/overview/working-with-objects/kubernetes-objects/).

In Onyxia we use the `values.schema.json` file to know what options should be displayed to the user at [the service configuration step](https://user-images.githubusercontent.com/6702424/177571819-f2e1b4ef-ecd1-479b-a5a1-658d87d7c7c0.png) and what default value Onyxia should inject.

![https://helm.sh/docs/topics/charts/#the-chart-file-structure](<../.gitbook/assets/image (6).png>)

Let's consider a sample of the `values.schema.json` of the InseeFrLab/helm-charts-datascience's Jupyter chart:

<pre class="language-javascript" data-title="values.schema.json"><code class="lang-javascript">"git": {
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
</strong><strong>                "overwriteDefaultWith": "git.name"
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
</strong><strong>                "overwriteDefaultWith": "git.email"
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
</strong><strong>                "overwriteDefaultWith": "git.credentials_cache_duration"
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
</strong><strong>                "overwriteDefaultWith": "git.token"
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

If the user took the time to fill its profile information, [onyxia-web](https://github.com/InseeFrLab/onyxia-web) know what is the Git **username**, **email** and **personal access token** of the user.

![The onyxia user profile](<../.gitbook/assets/image (7).png>)

[Here](https://github.com/InseeFrLab/onyxia/blob/main/web/src/core/ports/OnyxiaApi/XOnyxia.ts) is defined the structure of the context that you can use in the `overwriteDefaultWith` field:

```typescript
export type XOnyxiaParams = {
    /**
     * This is where you can reference values from the onyxia context so that they
     * are dynamically injected by the Onyxia launcher.
     *
     * Examples:
     * "overwriteDefaultWith": "user.email"
     * "overwriteDefaultWith": "{{project.id}}-{{k8s.randomSubdomain}}.{{k8s.domain}}"
     */
    overwriteDefaultWith?: string;
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
    kaggleApiToken: string | undefined;
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
    };
    proxyInjection:
        | {
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

You can also concatenate string values using [mustache](https://mustache.github.io/) syntax.

```javascript
"hostname": {
  "type": "string",
  "form": true,
  "title": "Hostname",
  "x-onyxia": {
    "overwriteDefaultWith": "{{project.id}}-{{k8s.randomSubdomain}}.{{k8s.domain}}"
  }
}
```

#### Defining region scoped resources limit

You probably want to be able to define a limit to the amount of resources a user can request when launching a service.

It's possible to do it at the catalog level but it's best to enable the person who is deploying Onyxia to define boundaries for his deployment regions.

This is the purpose of the `x-onyxia` param `useRegionSliderConfig`

<pre class="language-yaml" data-title="onyxia/values.yaml"><code class="lang-yaml">onyxia:

  web:
    ...
    
  api:
    ...
    regions:
      [
        {
          "id": "paris",
          "name": "Kubernetes DG Insee",
          "services": {
            "defaultConfiguration": {
<strong>              "sliders": {
</strong>                "cpu": {
                  "sliderMin": 100,
                  "sliderMax": 80000,
                  "sliderStep": 100,
                  "sliderUnit": "m"
                },
                "memory": {
                  "sliderMin": 1,
                  "sliderMax": 400,
                  "sliderStep": 1,
                  "sliderUnit": "Gi"
                },
<strong>                "gpu": {
</strong><strong>                  "sliderMin": 1,
</strong><strong>                  "sliderMax": 4,
</strong><strong>                  "sliderStep": 1,
</strong><strong>                  "sliderUnit": ""
</strong><strong>                },
</strong>                "disk": {
                  "sliderMin": 1,
                  "sliderMax": 100,
                  "sliderStep": 1,
                  "sliderUnit": "Gi"
                }
              },
              "resources": {
                "cpuRequest": "100m",
                "cpuLimit": "40000m",
                "memoryRequest": "1Gi",
                "memoryLimit": "200Gi",
                "disk": "10Gi",
<strong>                "gpu": "1"
</strong>              }
            }
          }
        }
      ]
</code></pre>

<pre class="language-json" data-title="values.schema.json"><code class="lang-json">{
  "$schema": "http://json-schema.org/schema#",
  "type": "object",
  "properties": {
    "resources": {
      "description": "Your service will have at least the requested resources and never more than its limits. No limit for a resource and you can consume everything left on the host machine.",
      "type": "object",
      "properties": {
        "limits": {
          "description": "max resources",
          "type": "object",
          "properties": {
            "nvidia.com/gpu": {
              "description": "GPU to allocate to this instance. This is also requested",
              "type": "string",
<strong>              "default": "0", // Will be overwritten by "1"
</strong>              "render": "slider",
<strong>              "sliderMin": 0, // Will be overwritten by 1
</strong><strong>              "sliderMax": 3, // Will be overwritten by 4
</strong><strong>              "sliderStep": 1, // Will be overwritten by 1
</strong><strong>              "sliderUnit": "", // Will be overwritten by ""
</strong><strong>              "x-onyxia": {
</strong><strong>                "overwriteDefaultWith": "region.resources.gpu",
</strong><strong>                "useRegionSliderConfig": "gpu" // 👀
</strong><strong>              }
</strong>            },
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
              "sliderRangeId": "cpu",
              "x-onyxia": {
                "overwriteDefaultWith": "region.resources.cpuLimit",
                "useRegionSliderConfig": "cpu"
              }
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
              "sliderRangeId": "memory",
              "x-onyxia": {
                "overwriteDefaultWith": "region.resources.memoryLimit",
                "useRegionSliderConfig": "memory"
              }
            }
          }
        }
      }
    }
  }
}
</code></pre>

You now have all the relevent information to submit PR on the existing catalogs or even to create your own.

Remember that a helm chart repository is nothing more than a GitHub repo with a special [github Action setup](https://github.com/marketplace/actions/helm-deploy) to publish the charts on GitHub Pages.

If you are looking for a repo to start from have a look at [this one](https://github.com/codegouvfr/helm-charts), it has a directory where you can put the icons of your services.
