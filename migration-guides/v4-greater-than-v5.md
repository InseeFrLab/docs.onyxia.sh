# ⬆ v4 -> v5

The primary breaking change in this release pertains to Keycloak configuration. With this update, you're no longer limited to using Keycloak; any OIDC-compliant identity provider is now supported.\
To accommodate this new feature, you'll need to make some adjustments to the configuration of your Onyxia instance.

{% hint style="info" %}
You don't need to specify the `issuerURI` in multiple locations as we have done here.\
If you're using just one identity server (You have only one Keycloak server for example), you can set the `issuerURI` solely in `api->env->oidc.issuer-uri`.
{% endhint %}

{% code title="onyxia-values.yaml" %}
```diff
onyxia:
   web:
     env:
-      KEYCLOAK_URL: https://auth.lab.sspcloud.fr/auth
-      KEYCLOAK_REALM: sspcloud
   api:
     env:
-      keycloak.resource: onyxia
-      keycloak.realm: sspcloud
-      keycloak.auth-server-url: https://auth.lab.sspcloud.fr/auth
-      keycloak.ssl-required: external
-      keycloak.public-client: "true"
-      keycloak.enable-basic-auth: "true"
-      keycloak.bearer-only: "true"
+      oidc.issuer-uri: "https://auth.lab.sspcloud.fr/auth/realms/sspcloud"
+      oidc.clientID: "onyxia"
+      oidc.audience: "onyxia"
       authentication.mode: "openidconnect"
     regions: 
       [
         {
           "id": "paris",
           "services": {
-              "authenticationMode": "admin",
+              "authenticationMode": "serviceAccount",
               "k8sPublicEndpoint": {
                 "URL": "https://apiserver.kub.sspcloud.fr",
-                "keycloakParams": {
-                  "URL": "https://auth.lab.sspcloud.fr/auth",
-                  "realm": "sspcloud",
-                  "clientId": "onyxia"
-                },
+                "oidcConfiguration": {
+                  "issuerURI": "https://auth.lab.sspcloud.fr/auth/realms/sspcloud",
+                  "clientID": "onyxia-k8s-apiserver",
+                }
               }
             },
           "data": {
             "S3": {
-              "keycloakParams": {
-                "URL": "https://auth.lab.sspcloud.fr/auth",
-                "realm": "sspcloud",
-                "clientId": "onyxia-minio",
-              }
+              "oidcConfiguration": {
+                "issuerURI": "https://auth.lab.sspcloud.fr/auth/realms/sspcloud",
+                "clientID": "onyxia-minio",
+              }
             }
          },
          "vault": {
              "URL": "https://vault.lab.sspcloud.fr",
-             "keycloakParams": {
-               "URL": "https://auth.lab.sspcloud.fr/auth",
-               "realm": "sspcloud",
-               "clientId": "onyxia-vault",
-             }
+             "oidcConfiguration": {
+               "issuerURI": "https://auth.lab.sspcloud.fr/auth/realms/sspcloud",
+               "clientID": "onyxia-vault"
+             }
         }
       }
     ]
```
{% endcode %}
