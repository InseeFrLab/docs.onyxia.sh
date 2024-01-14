---
description: Using Keycloak to enable user authentication
---

# 🔑 User authentication

Let's setup Keycloak to enable users to create account and login to our Onyxia.

{% hint style="success" %}
Note that in this instalation guide we make you use Keycloak but you can use any identity server that is Open ID Connect compliant.
{% endhint %}

For deploying our Keycloak we use [codecentric's Keycloakx Helm chart](https://github.com/codecentric/helm-charts/tree/master/charts/keycloakx) and [Bitnami's PostgressSQL Helm chart](https://github.com/bitnami/charts/tree/main/bitnami/postgresql). &#x20;

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
```



```bash
helm repo add codecentric https://codecentric.github.io/helm-charts

DOMAIN=my-domain.net
POSTGRESQL_PASSWORD=xxxxx #Replace by a strong password, you will never need it.
# Credentials for logging to https://auth.lab.$DOMAIN/auth
KEYCLOAK_USER=admin
KEYCLOAK_PASSWORD=yyyyyy 

cat << EOF > ./keycloak-values.yaml
image:
  # We use the legacy variant of the image until codecentric update it's helm chart
  tag: "19.0.3-legacy"
replicas: 1
extraInitContainers: |
  - name: realm-ext-provider
    image: curlimages/curl
    imagePullPolicy: IfNotPresent
    command:
      - sh
    args:
      - -c
      - |
        # There is a custom theme published alongside every onyxia-web release
        # The version of the Keycloak theme and the version of onyxia-web don't need 
        # to match but you should update the theme from time to time.  
        # https://github.com/InseeFrLab/onyxia-web/releases
        curl -L -f -S -o /extensions/onyxia.jar https://github.com/InseeFrLab/onyxia-web/releases/download/v2.29.4
/keycloak-theme.jar
    volumeMounts:
      - name: extensions
        mountPath: /extensions
extraVolumeMounts: |
  - name: extensions
    mountPath: /opt/jboss/keycloak/standalone/deployments
extraVolumes: |
  - name: extensions
    emptyDir: {}
extraEnv: |
  - name: ONYXIA_RESOURCES_ALLOWED_ORIGINS
    value: "https://onyxia.$DOMAIN, http://localhost, http://127.0.0.1"
# The following values are transported automatically from your
# Onyxia instance over to the login pages via url parameters, however,
# if you want to prevent thoses value from being potentially
# hijacked by a third party you can hard code them here.  
# - name: ONYXIA_HEADER_TEXT_BOLD
#   value: "My organization"
# - name: ONYXIA_HEADER_TEXT_FOCUS
#   value: "Datalab"
# - name: ONYXIA_PALETTE_OVERRIDE
#   value: "{}"
# - name: ONYXIA_TAB_TITLE
#   value: "Onyxia"
  - name: KEYCLOAK_USER
    value: $KEYCLOAK_USER
  - name: KEYCLOAK_PASSWORD
    value: $KEYCLOAK_PASSWORD
  - name: JGROUPS_DISCOVERY_PROTOCOL
    value: kubernetes.KUBE_PING
  - name: KUBERNETES_NAMESPACE
    valueFrom:
     fieldRef:
       apiVersion: v1
       fieldPath: metadata.namespace
  - name: KEYCLOAK_STATISTICS
    value: "true"
  - name: CACHE_OWNERS_COUNT
    value: "2"
  - name: CACHE_OWNERS_AUTH_SESSIONS_COUNT
    value: "2"
  - name: PROXY_ADDRESS_FORWARDING
    value: "true"
  - name: JAVA_OPTS
    value: >-
      -Dkeycloak.profile=preview -XX:+UseContainerSupport -XX:MaxRAMPercentage=50.0 -Djava.net.preferIPv4Stack=true -Djava.awt.headless=true 
ingress:
  enabled: true
  servicePort: http
  annotations:
    kubernetes.io/ingress.class: nginx
    ## Resolve HTTP 502 error using ingress-nginx:
    ## See https://www.ibm.com/support/pages/502-error-ingress-keycloak-response
    nginx.ingress.kubernetes.io/proxy-buffer-size: 128k
  rules:
    - host: "auth.lab.$DOMAIN"
      paths:
        - path: /
          pathType: Prefix
  tls:
    - hosts:
        - auth.lab.$DOMAIN
postgresql:
  postgresqlPassword: $POSTGRESQL_PASSWORD
EOF

helm install keycloak codecentric/keycloak -f keycloak-values.yaml
```

You can now login to the **administration console** of **https://auth.lab.my-domain.net** and login using the credentials you have defined with `KEYCLOAK_USER` and `KEYCLOAK_PASSWORD`.

1. Create a realm called "datalab" (or something else), go to **Realm settings**
   1. On the tab General
      1. _User Profile Enabled_: **On**
   2. On the tab **login**
      1. _User registration_: **On**
      2. _Forgot password_: **On**
      3. _Remember me_: **On**
   3. On the tab **email,** we give an example with \*\*\*\* [AWS SES](https://aws.amazon.com/ses/), if you don't have a SMTP server at hand you can skip this by going to **Authentication** (on the left panel) -> Tab **Required Actions** -> Uncheck "set as default action" **Verify Email**. Be aware that with email verification disable, anyone will be able to sign up to your service.
      1. _From_: **noreply@lab.my-domain.net**
      2. _Host_: **email-smtp.us-east-2.amazonaws.com**
      3. _Port_: **465**
      4. _Authentication_: **enabled**
      5. _Username_: **\*\*\*\*\*\*\*\*\*\*\*\*\*\***
      6. _Password_: **\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\***
      7. When clicking "save" you'll be asked for a test email, you have to provide one that correspond **to a pre-existing user** or you will get a silent error and the credentials won't be saved.
   4. On the tab Themes
      1. _Login theme_: **onyxia-web** (you can also select the login theme on a per client basis)
      2. _Email theme_: **onyxia-web**
   5. On the tab **Localization**
      1. _Internationalization_: **Enabled**
      2. _Supported locales_: \<Select the languages you wish to support>
   6. On the tab **Token**
      1. SSO Session Idle: 14 days - This setting and the following two are so that when the user click "remember me" when he logs in, he dosen't have to loggin again for the next two weeks.
      2. SSO Session Idle Remember Me: 14 days
      3. SSO Session Max Remember Me: 14 days
2. Create a client called "onyxia"
   1. _Root URL_: **https://onyxia.my-domain.net/**
   2. _Valid redirect URIs_: **https://onyxia.my-domain.net/\***
   3. _Web origins_: **\***
   4. Login theme: **onyxia-web**
3. In **Authentication** (on the left panel) -> Tab **Required Actions** enable and set as default action **Therms and Conditions.**

Now you want to ensure that the username chosen by your users complies with Onyxia requirement (only alphanumerical characters) and define a list of email domain allowed to register to your service.

Go to **Realm Settings** (on the left panel) -> Tab **User Profile** (this tab shows up only if User Profile is enabled in the General tab and you can enable user profile only if you have started Keycloak with `-Dkeycloak.profile=preview)` -> **JSON Editor**.

Now you can edit the file as suggested in the following DIFF snippet. Be mindful that in this example we only allow emails @gmail.com and @hotmail.com to register you want to edit that.

```diff
{
  "attributes": [
    {
      "name": "username",
      "displayName": "${username}",
      "validations": {
        "length": {
          "min": 3,
          "max": 255
        },
+       "pattern": {
+         "error-message": "${alphanumericalCharsOnly}",
+         "pattern": "^[a-zA-Z0-9]*$"
+       },
        "username-prohibited-characters": {}
      }
    },
    {
      "name": "email",
      "displayName": "${email}",
      "validations": {
        "email": {},
+       "pattern": {
+         "pattern": "^[^@]+@([^.]+\\.)*((gmail\\.com)|(hotmail\\.com))$"
+       },
        "length": {
          "max": 255
        }
      }
    },
    {
      "name": "firstName",
      "displayName": "${firstName}",
      "required": {
        "roles": [
          "user"
        ]
      },
      "permissions": {
        "view": [
          "admin",
          "user"
        ],
        "edit": [
          "admin",
          "user"
        ]
      },
      "validations": {
        "length": {
          "max": 255
        },
        "person-name-prohibited-characters": {}
      }
    },
    {
      "name": "lastName",
      "displayName": "${lastName}",
      "required": {
        "roles": [
          "user"
        ]
      },
      "permissions": {
        "view": [
          "admin",
          "user"
        ],
        "edit": [
          "admin",
          "user"
        ]
      },
      "validations": {
        "length": {
          "max": 255
        },
        "person-name-prohibited-characters": {}
      }
    }
  ]
}
```

Now our Keycloak server is fully configured we just need to update our Onyxia deployment to let it know about it.

Update the `onyxia-values.yaml` file that you created previously, don't forget to replace all the occurence of **my-domain.net** by your actual domain.

Don't forget as well to remplace the terms of services of the [sspcloud](https://www.sspcloud.fr) by your own terms of services. CORS should be enabled on those `.md` links (`Access-Control-Allow-Origin: *`).

```diff
+serviceAccount:
+  clusterAdmin: true
 ingress:
   enabled: true
   annotations:
     kubernetes.io/ingress.class: nginx
   hosts:
     - host: onyxia.my-domain.net
 web:
+  env:
+    TERMS_OF_SERVICES: |
+      { 
+        "en": "https://www.sspcloud.fr/tos_en.md", 
+        "fr": "https://www.sspcloud.fr/tos_fr.md" 
+      }
 api:
   env:
+    authentication.mode: openidconnect
+    oidc.issuer-uri: "https://auth.lab.my-domain.net/auth/realms/datalab"
+    oidc.clientID: "onyxia"
+    oidc.audience: "onyxia"
   regions:
     [
        {
           "id":"demo",
           "name":"Demo",
           "description":"This is a demo region, feel free to try Onyxia !",
           "services":{
              "type":"KUBERNETES",
-             "singleNamespace": true,
+             "singleNamespace": false,
              "namespacePrefix":"user-",
              "usernamePrefix":"oidc-",
              "groupNamespacePrefix":"projet-",
              "groupPrefix":"oidc-",
              "authenticationMode":"serviceAccount",
              "expose":{
                 "domain":"lab.my-domain.net"
              },
              "monitoring":{
                 "URLPattern":"todo"
              },
              "initScript":"https://inseefrlab.github.io/onyxia/onyxia-init.sh"
           }
        }
     ]
```

Now that you have updated `onyxia-values.yaml` restart onyxia-web with the new configuration.

```bash
helm upgrade onyxia inseefrlab/onyxia -f onyxia-values.yaml
```

Now your users should be able to create account, log-in, and start services on their own Kubernetes namespace.
