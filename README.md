# ⬆ Migration v0 -> v2

The vault configuration, like S3 configurations, are now configured in the region and no more as environement variables passed to onyxia-web.

* The vault configuration, like S3 configurations, are now configured in the region and no longer as environement variables passed to onyxia-web.
* The highlighted charts (the packages that should be displayed first in the UI) are now configured in the catalogs and no longer as an environment variable passed to onyxia-web.

<pre class="language-diff"><code class="lang-diff"> onyxia:
   ui:
     image:
       name: inseefrlab/onyxia-web
-      version: 0.59.1
+      version: 2.0.0
     env:
-      VAULT_URL: https://vault.lab.sspcloud.fr
<strong>-      VAULT_KV_ENGINE=onyxia-kv
</strong>-      VAULT_ROLE=onyxia-user
-      VAULT_KEYCLOAK_URL=https://auth.my-domain/auth
-      VAULT_KEYCLOAK_REALM=myrealm
-      VAULT_KEYCLOAK_CLIENT_ID=vault

-      HIGHLIGHTED_PACKAGES: |
-        ["jupyter", "rstudio", "vscode", "ubuntu", "postgres"]

   api:
     image:
       name: inseefrlab/onyxia-api
       version: v0.15
     regions: 
       [
         {
           "id": "paris",
           "name": "Kubernetes DG Insee",
           "description": "Region principale. Plateforme hébergée sur les serveurs de la direction générale de l'INSEE à Montrouge.",
           "services": { ... },
           "data": { ... },
+          "vault": { 
+                "URL": "https://auth.my-domain/auth",
+                "kvEngine": "onyxia-kv",
+                "role": "onyxia-user",
+                "keycloakParams": {
+                  "URL":"https://auth.my-domain/auth",
+                  "realm":"myrealm",
+                  "clientId":"vault"
+                }
+          }
         }
       ]
    catalogs: 
      [
        {
          "id": "inseefrlab-helm-charts-datascience",
          ...
+         "highlightedCharts": ["rstudio","jupyter","vscode","ubuntu","postgresql"]
         }
       ]</code></pre>

### Highlighted charts

<pre class="language-diff"><code class="lang-diff">onyxia:
  serviceAccount:
    create: true
    clusterAdmin: true
  ingress:
    enabled: true
    annotations:
      kubernetes.io/ingress.class: nginx
    hosts:
      - host: onyxia.kub.sspcloud.fr
      - host: onyxia.lab.sspcloud.fr
      - host: datalab.sspcloud.fr
    tls:
      - hosts:
          - onyxia.kub.sspcloud.fr
          - onyxia.lab.sspcloud.fr
          - datalab.sspcloud.fr
  ui:
    replicaCount: 2
    image:
      name: inseefrlab/onyxia-web
      version: 1.0.1
    nodeSelector:
      infra: "true"
    tolerations:
      - key: "infra"
        operator: "Exists"
    env:
      KEYCLOAK_URL: https://auth.lab.sspcloud.fr/auth
      KEYCLOAK_REALM: sspcloud
      HEADER_ORGANIZATION: SSP Cloud
      TERMS_OF_SERVICES: |
        { "en": "https://www.sspcloud.fr/tos_en.md", "fr": "https://www.sspcloud.fr/tos_fr.md" }
<strong>      HIGHLIGHTED_PACKAGES: |
</strong>        ["jupyter","jupyter-gpu","jupyter-pyspark","jupyter-r","rstudio","rstudio-gpu","rstudio-sparkr","vscode","vscode-gpu","ubuntu","postgres"]
      DESCRIPTION: Plateforme mutualisée de services de traitement des données statistiques et de datascience
      HEADER_LINKS: |
        [
          { 
            "label": { "en": "Trainings", "fr": "Formations", "zh-CN":"教程" }, 
            "iconId": "training", 
            "url": "https://www.sspcloud.fr/formation" 
          }, 
          { 
            "label": { "en": "Documentation", "zh-CN":"文档" },
            "iconId": "language", 
            "url": "https://docs.sspcloud.fr" 
          }
        ]
  api:
    replicaCount: 3
    image:
      name: inseefrlab/onyxia-api
      version: v0.15
    nodeSelector:
      infra: "true"
    tolerations:
      - key: "infra"
        operator: "Exists"
    env:
      security.cors.allowed_origins: "*" # Todo restore when we have another preview
      keycloak.resource: onyxia
      keycloak.realm: sspcloud
      keycloak.auth-server-url: https://auth.lab.sspcloud.fr/auth
      keycloak.ssl-required: external
      keycloak.public-client: "true"
      keycloak.enable-basic-auth: "true"
      keycloak.bearer-only: "true"
      authentication.mode: "openidconnect"
      springdoc.swagger-ui.oauth.clientId: onyxia
      catalog.refresh.ms: "300000"
      springdoc.swagger-ui.path: "/api"
<strong>    catalogs: 
</strong>      [
        {
          "id": "ide",
          "name": "Services interactifs",
          "description": "Services for datascientists.",
          "maintainer": "innovation@insee.fr",
          "location": "https://inseefrlab.github.io/helm-charts-datascience-internal",
          "status": "PROD",
          "highlightedCharts": ["jupyter","jupyter-gpu","jupyter-pyspark","jupyter-r","rstudio","rstudio-gpu","rstudio-sparkr","vscode","vscode-gpu","ubuntu","postgres"],
          "type": "helm"
        },
        {
          "id": "inseefrlab-helm-charts-datascience",
          "name": "Inseefrlab datascience",
          "description": "Services for datascientists.",
          "maintainer": "innovation@insee.fr",
          "location": "https://inseefrlab.github.io/helm-charts-datascience",
          "status": "PROD",
          "highlightedCharts": ["rstudio","jupyter","vscode","ubuntu","postgresql"],
          "type": "helm"
        },
        {
          "id": "inseefrlab-helm-charts-trainings",
          "name": "Inseefrlab trainings",
          "description": "Trainings for datascientists.",
          "maintainer": "innovation@insee.fr",
          "location": "https://inseefrlab.github.io/helm-charts-trainings",
          "status": "TEST",
          "type": "helm"
        },
        {
          "id": "inseefrlab-helm-charts-shiny-apps",
          "name": "Inseefrlab R Shiny apps",
          "description": "R Shiny apps as services.",
          "maintainer": "innovation@insee.fr",
          "location": "https://inseefrlab.github.io/helm-charts-shiny-apps",
          "status": "PROD",
          "type": "helm"
        },
      ]
    regions: 
      [
        {
          "id": "paris",
          "name": "Kubernetes DG Insee",
          "description": "Region principale. Plateforme hébergée sur les serveurs de la direction générale de l'INSEE à Montrouge.",
          "services": {
            "type": "KUBERNETES",
            "k8sPublicEndpoint": { 
                "URL": "https://apiserver.kub.sspcloud.fr",
                "keycloakParams": {
                  "URL":"https://auth.lab.sspcloud.fr/auth",
                  "realm":"sspcloud",
                  "clientId":"onyxia"
                }
            },
            "singleNamespace": false,
            "namespacePrefix": "user-",
            "usernamePrefix": "oidc-",
            "groupNamespacePrefix": "projet-",
            "groupPrefix": "oidc-",
            "authenticationMode": "admin",
            "expose": { "domain": "kub.sspcloud.fr" },
            "monitoring": { "URLPattern": "https://grafana.lab.sspcloud.fr/d/kYYgRWBMz/users-services?orgId=1&#x26;refresh=5s&#x26;var-namespace=$NAMESPACE&#x26;var-instance=$INSTANCE" },
            "cloudshell": {
              "catalogId": "inseefrlab-helm-charts-datascience",
              "packageName": "cloudshell"
            },
            "quotas": {
                "enabled": true,
                "allowUserModification": true,
                "default": {
                  "requests.storage": "2Ti",
                  "count/pods": "200"
                }
              },
              "defaultConfiguration": {
                "ipprotection": true,
                "networkPolicy": true,
                "from": [
                  {
                    "ipBlock": {
                      "cidr": "10.233.103.0/32"
                    }
                  },
                  {
                    "ipBlock": {
                      "cidr": "10.233.111.0/32"
                    }
                  }
                ],
                "startupProbe": {
                  "failureThreshold": 60,
                  "initialDelaySeconds": 10,
                  "periodSeconds": 10,
                  "successThreshold": 1,
                  "timeoutSeconds": 5
                },
                "kafka":{
                  "URL": "kafka-0.kafka-headless.atlas:9092,kafka-1.kafka-headless.atlas:9092,kafka-2.kafka-headless.atlas:9092",
                  "topicName":"hive-meta"
                }
              },
            "initScript": "https://InseeFrLab.github.io/onyxia/onyxia-init.sh"
          },
          "data": { 
             "S3": { 
                "type": "minio",
                "URL": "https://minio.lab.sspcloud.fr",
                "region": "us-east-1",
                "bucketPrefix": "",
                "groupBucketPrefix": "projet-",
                "bucketClaim": "preferred_username",
                "defaultDurationSeconds": 86400,
                "keycloakParams": {
                  "URL":"https://auth.lab.sspcloud.fr/auth",
                  "realm":"sspcloud",
                  "clientId":"onyxia"
                },
                "monitoring": { "URLPattern": "https://grafana.lab.sspcloud.fr/d/PhCwEJkMz/user-s3-storage?orgId=1&#x26;var-username=$BUCKET_ID" } 
            },
            "atlas": { 
                "URL": "https://atlas.lab.sspcloud.fr",
                "keycloakParams": {
                  "URL":"https://auth.lab.sspcloud.fr/auth",
                  "realm":"sspcloud",
                  "clientId":"atlas"
                }
            } 
          },
          "vault": { 
                "URL": "https://vault.lab.sspcloud.fr",
                "kvEngine": "onyxia-kv",
                "role": "onyxia-user"
          },
          "auth": { "type": "openidconnect" },
          "location": { "lat": 48.8164, "long": 2.3174, "name": "Montrouge (France)" }
        }
      ]</code></pre>

