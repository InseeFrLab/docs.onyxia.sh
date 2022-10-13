# ⬆ Migration v0 -> v2

The vault configuration, like S3 configurations, are now configured in the region and no more as environement variables passed to onyxia-web.

* The vault configuration, like S3 configurations, are now configured in the region and no longer as environement variables passed to onyxia-web.
* The highlighted charts (the packages that should be [displayed first in the UI](https://user-images.githubusercontent.com/6702424/192175837-068ca4cd-9083-4b54-8cd7-5194bdb1e23c.png)) are now configured in the catalogs and no longer as an environment variable passed to onyxia-web.

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
