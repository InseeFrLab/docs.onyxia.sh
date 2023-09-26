---
description: Your Onyxia instance, today
---

# 🏁 Install

## Oneliner

TLDR. Here is how you can get an Onyxia instance running in a matter of seconds.

```bash
helm repo add onyxia https://inseefrlab.github.io/onyxia

cat << EOF > ./onyxia-values.yaml
ingress:
  enabled: true
  hosts:
    - host: datalab.my-domain.net
EOF

helm install onyxia onyxia/onyxia --version "4.0.2" -f onyxia-values.yaml
```

With this, you will obtain an instance operating in a degraded mode, which lacks features such as authentication, S3 explorer, secret management, etc. However, you will still have the capability to launch services from the catalog.

## Step by step installation guide

In this section, we will set up Onyxia from the ground up, along with all the associated technologies. This includes [MinIO](https://min.io/) for S3, [Keycloak](https://www.keycloak.org/) for OIDC, and [Vault](https://www.vaultproject.io/) for managing secrets.

### Provision a Kubernetes cluster

First you'll need a Kubernetes cluster. If you have one already you can skip this section.

{% tabs %}
{% tab title="Provisioning a cluster on AWS, GCP or Azure" %}
[Hashicorp](https://www.hashicorp.com/) maintains great tutorials for [terraforming](https://www.terraform.io/) Kubernetes clusters on [AWS](https://aws.amazon.com/what-is-aws/), [GCP](https://cloud.google.com/) or [Azure](https://acloudguru.com/videos/acg-fundamentals/what-is-microsoft-azure).

Pick one of the three and follow the guide.

You can stop after the [configure kubectl section](https://learn.hashicorp.com/tutorials/terraform/eks#configure-kubectl).

{% embed url="https://developer.hashicorp.com/terraform/tutorials/kubernetes/eks" %}

{% embed url="https://developer.hashicorp.com/terraform/tutorials/kubernetes/gke?in=terraform%2Fkubernetes" %}

{% embed url="https://developer.hashicorp.com/terraform/tutorials/kubernetes/aks?in=terraform%2Fkubernetes" %}

**Ingress controller**

Deploy an ingress controller on your cluster:

{% hint style="warning" %}
The following command is [for AWS](https://kubernetes.github.io/ingress-nginx/deploy/#aws).

For GCP use [this command](https://kubernetes.github.io/ingress-nginx/deploy/#gce-gke).

For Azure use [this command](https://kubernetes.github.io/ingress-nginx/deploy/#azure).
{% endhint %}

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.2.0/deploy/static/provider/aws/deploy.yaml
```

**DNS**

Let's assume you own the domain name **my-domain.net**, for the rest of the guide you should replace **my-domain.net** by a domain you actually own.

Now you need to get the external address of your cluster, run the command

```bash
kubectl get services -n ingress-nginx
```

and write down the `External IP` assigned to the `LoadBalancer`.

Depending on the cloud provider you are using it can be an IPv4, an IPv6 or a domain. On AWS for example, it will be a domain like **xxx.elb.eu-west-1.amazonaws.com**.

If you see `<pending>`, wait a few seconds and try again.

Once you have the address, create the following DNS records:

```dns-zone-file
onyxia.my-domain.net CNAME xxx.elb.eu-west-1.amazonaws.com. 
*.lab.my-domain.net  CNAME xxx.elb.eu-west-1.amazonaws.com. 
```

If the address you got was an IPv4 (`x.x.x.x`), create a `A` record instead of a CNAME.

If the address you got was ans IPv6 (`y:y:y:y:y:y:y:y`), create a `AAAA` record.

**https://onyxia.my-domain.net** will be the URL for your instance of Onyxia. The URL of the services created by Onyxia are going to look like: **https://\<something>.lab.my-domain.net**

{% hint style="info" %}
You can customise "**onyxia**" and "**lab**" to your liking, for example you could chose **datalab.my-domain.net** and **\*.kub.my-domain.net**.
{% endhint %}

**SSL**

In this section we will obtain a TLS certificate issued by [LetsEncrypt](https://letsencrypt.org/) using the [certbot](https://certbot.eff.org/) commend line tool then get our ingress controller to use it.

If you are already familiar with `certbot` you're probably used to run it on a remote host via SSH. In this case you are expected to run it on your own machine, we'll use the DNS chalenge instead of the HTTP chalenge.

```bash
brew install certbot #On Mac, lookup how to install certbot for your OS

#Because we need a wildcard certificate we have to complete the DNS callange.  
sudo certbot certonly --manual --preferred-challenges dns

# When asked for the domains you wish to optains a certificate for enter:
#   onyxia.my-domain.net *.lab.my-domain.net
```

{% hint style="info" %}
The obtained certificate needs to be renewed every three month.

To avoid the burden of having to remember to re-run the `certbot` command periodically you can setup [cert-manager](https://cert-manager.io/) and configure a [DNS01 challenge provider](https://cert-manager.io/docs/configuration/acme/dns01/) on your cluster but that's out of scope for Onyxia.

You may need to delegate your DNS Servers to one of the supported [DNS service provider](https://cert-manager.io/docs/configuration/acme/dns01/#supported-dns01-providers).
{% endhint %}

Now we want to create a Kubernetes secret containing our newly obtained certificate:

```bash
DOMAIN=my-domain.net
sudo kubectl create secret tls onyxia-tls \
    -n ingress-nginx \
    --key /etc/letsencrypt/live/onyxia.$DOMAIN/privkey.pem \
    --cert /etc/letsencrypt/live/onyxia.$DOMAIN/fullchain.pem
```

Lastly, we want to tell our ingress controller to use this TLS certificate, to do so run:

```bash
kubectl edit deployment ingress-nginx-controller -n ingress-nginx
```

This command will open your configured text editor, go to line `56` and add:

```
      - --default-ssl-certificate=ingress-nginx/onyxia-tls
```

![](<.gitbook/assets/image (1) (2).png>)
{% endtab %}

{% tab title="Test on your machine" %}
If you are on a Mac or Window computer you can install [Docker desktop](https://www.docker.com/products/docker-desktop/) then enable Kubernetes.

![Enable Kubernetes in Docker desktop](<.gitbook/assets/image (5) (1).png>)

{% hint style="info" %}
Docker desktop isn't available on Linux, you can use [Kind](https://kind.sigs.k8s.io/) instead.
{% endhint %}

**Port Forwarding**

You'll need to [forward the TCP ports 80 and 443 to your local machine](https://user-images.githubusercontent.com/6702424/174459930-23fb577c-11a2-49ef-a082-873f4139aca1.png). It's done from the administration panel of your domestic internet Box. If you're on a corporate network, no luck for you I'm afraid.

**DNS**

Let's assume you own the domain name **my-domain.net,** for the rest of the guide you should replace **my-domain.net** by a domain you actually own.

Get [your internet box routable IP](http://monip.org/) and create the following DNS records:

```dns-zone-file
onyxia.my-domain.net A <YOUR_IP>
*.lab.my-domain.net  A <YOUR_IP>
```

{% hint style="success" %}
If you have DDNS domain you can create `CNAME` instead example:

```
onyxia.my-domain.net CNAME jhon-doe-home.ddns.net.
*.lab.my-domain.net  CNAME jhon-doe-home.ddnc.net.
```
{% endhint %}

_**https://onyxia.my-domain.net**_ will be the URL for your instance of Onyxia.

The URL of the services created by Onyxia are going to look like: _**https://xxx.lab.my-domain.net**_

{% hint style="info" %}
You can customise "**onyxia**" and "**lab**" to your liking, for example you could chose **datalab.my-domain.net** and **\*.kub.my-domain.net**.
{% endhint %}

**SSL**

In this section we will obtain a TLS certificate issued by [LetsEncrypt](https://letsencrypt.org/) using the [certbot](https://certbot.eff.org/) commend line tool.

```bash
brew install certbot #On Mac, lookup how to install certbot for your OS

# Because we need a wildcard certificate we have to complete the DNS callange.  
sudo certbot certonly --manual --preferred-challenges dns

# When asked for the domains you wish to optains a certificate for enter:
#   onyxia.my-domain.net *.lab.my-domain.net
```

{% hint style="info" %}
The obtained certificate needs to be renewed every three month.

To avoid the burden of having to remember to re-run the `certbot` command periodically you can setup [cert-manager](https://cert-manager.io/) and configure a [DNS01 challenge provider](https://cert-manager.io/docs/configuration/acme/dns01/) on your cluster but that's out of scope for Onyxia.

You may need to delegate your DNS Servers to one of the supported [DNS service provider](https://cert-manager.io/docs/configuration/acme/dns01/#supported-dns01-providers).
{% endhint %}

Now we want to create a Kubernetes secret containing our newly obtained certificate:

```bash
kubectl create namespace ingress-nginx
DOMAIN=my-domain.net
sudo kubectl create secret tls onyxia-tls \
    -n ingress-nginx \
    --key /etc/letsencrypt/live/onyxia.$DOMAIN/privkey.pem \
    --cert /etc/letsencrypt/live/onyxia.$DOMAIN/fullchain.pem
```

**Ingress controller**

We'll install [ingress-nginx](https://kubernetes.github.io/ingress-nginx/) in our cluster ~~but any other ingress controller will do~~.

```bash
cat << EOF > ./ingress-nginx-values.yaml
controller:
  extraArgs:
    default-ssl-certificate: "ingress-nginx/onyxia-tls"
EOF

helm install ingress-nginx ingress-nginx \
    --repo https://kubernetes.github.io/ingress-nginx \
    --namespace ingress-nginx \
    -f ./ingress-nginx-values.yaml
```
{% endtab %}
{% endtabs %}

### Installing Onyxia using helm

In this section we assume that:

* You have a Kubernetes cluster and `kubectl` configured
* **onyxia.my-domain.net** and **\*.lab.my-domain.net** are pointing to your cluster's external address. **my-domain.net** being a domain that you own. You can customise "**onyxia**" and "**lab**" to your liking, for example you could chose **datalab.my-domain.net** and **\*.kub.my-domain.net**.
* You have an ingress controller configured with a default TLS certificate for **\*.lab.my-domain.net** and **onyxia.my-domain.net**.

{% hint style="warning" %}
As of today [the default service catalog](https://github.com/InseeFrLab/helm-charts-datascience) will only work with [ingress-nginx](https://kubernetes.github.io/ingress-nginx/).

This will be addressed in the near future.
{% endhint %}

{% hint style="success" %}
Through out this guide we make as if everything was instantaneous. In reality if you are testing on a small cluster you will need to wait several minutes after hitting `helm install` for the services to be ready.

Use `kubectl get pods` to see if your pods are up and ready.

<img src=".gitbook/assets/image (1).png" alt="" data-size="original">
{% endhint %}

<details>

<summary>(Optional) Make sure that your cluster is ready for Onyxia</summary>

To make sure that your Kubernetes cluster is correctly configured let's deploy a test web app on it before deploying Onyxia.

<img src=".gitbook/assets/image (8).png" alt="The hello world SPA deployed" data-size="original">

```bash
DOMAIN=my-domain.net

cat << EOF > ./test-spa-values.yaml
ingress:
  enabled: true
  annotations:
    kubernetes.io/ingress.class: nginx
  hosts:
    - host: test-spa.lab.$DOMAIN
EOF

helm repo add etalab https://etalab.github.io/helm-charts
helm install test-spa etalab/keycloakify-demo-app -f test-spa-values.yaml
echo "Navigate to https://test-spa.lab.$DOMAIN, see the Hello World"
helm uninstall test-spa
```

</details>

```bash
helm repo add onyxia https://inseefrlab.github.io/onyxia

DOMAIN=my-domain.net

cat << EOF > ./onyxia-values.yaml
ingress:
  enabled: true
  annotations:
    kubernetes.io/ingress.class: nginx
  hosts:
    - host: onyxia.$DOMAIN
api:
  env:
    security.cors.allowed_origins: "http://localhost:3000"
  regions: 
    [
       {
          "id":"demo",
          "name":"Demo",
          "description":"This is a demo region, feel free to try Onyxia !",
          "services":{
             "type":"KUBERNETES",
             "singleNamespace":true,
             "namespacePrefix":"user-",
             "usernamePrefix":"oidc-",
             "groupNamespacePrefix":"projet-",
             "groupPrefix":"oidc-",
             "authenticationMode":"admin",
             "expose":{
                "domain":"lab.$DOMAIN"
             },
             "monitoring":{
                "URLPattern":"todo"
             },
             "cloudshell":{
                "catalogId":"inseefrlab-helm-charts-datascience",
                "packageName":"cloudshell"
             },
             "initScript":"https://inseefrlab.github.io/onyxia/onyxia-init.sh"
          },
          "data":{
             "S3":{
                "URL":"todo",
                "monitoring":{
                   "URLPattern":"todo"
                }
             }
          },
          "auth":{
             "type":"openidconnect"
          },
          "location":{
             "lat":48.8164,
             "long":2.3174,
             "name":"Montrouge (France)"
          }
       }
    ]
EOF

helm install onyxia onyxia/onyxia -f onyxia-values.yaml
```

You can now access `https://onyxia.my-domain.net` and start services. Congratulations! 🥳

{% hint style="info" %}
You have the ability to customize the user interface (UI) of Onyxia through the provision of specific environment variables to the UI. For details on the available options, please consult the 'UI Customization' section of [this file](https://github.com/InseeFrLab/onyxia-web/blob/main/.env).

If you are unsure about how to supply these variables, refer to the later section of this guide where we discuss how to provide the KEYCLOAK\_\* parameters. You'll then be able to add your UI-related parameters alongside them.
{% endhint %}

### Enabling user authentication

At the moment there is no authentication process, everyone can access our platform and and start services.

Let's setup Keycloak to enable users to create account and login to our Onyxia.

<details>

<summary>Notes if you already have a Keycloak server</summary>

If you already have a Keycloak server it is up to you to pick from this guide what is rellevent to you.

Main takeway is that you probably want to load the Onyxia custom Keycloak theme and enable `-Dkeycloak.profile=preview` in order to be able to enforce that usernames are well formatted and define an accept list of email domains allowed to create account on your Onyxia instance.

You probably want to enable [Terms and Conditions as required actions](https://docs.keycloakify.dev/terms-and-conditions).

That out of the way, note that you can configure onyxia-web to integrate with your existing Keycloak server, you just need to set some dedicated environment variable in the `values.yaml` of the onyxia helm chart. Example:

```yaml
 web:
  env:
    # Available env are documented here: https://github.com/InseeFrLab/onyxia-web/blob/main/.env
    KEYCLOAK_URL: https://auth.lab.my-domain.net/auth
    KEYCLOAK_CLIENT_ID: onyxia
    KEYCLOAK_REALM: datalab
    JWT_EMAIL_CLAIM: email
    JWT_FAMILY_NAME_CLAIM: family_name
    JWT_FIRST_NAME_CLAIM: given_name
    JWT_USERNAME_CLAIM: preferred_username
    JWT_LOCALE_CLAIM: locale
```

</details>

For deploying our Keycloak we use [codecentric's helm chart](https://github.com/codecentric/helm-charts/tree/master/charts/keycloak).

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
+    KEYCLOAK_REALM: datalab
+    KEYCLOAK_URL: https://auth.lab.my-domain.net/auth
+    TERMS_OF_SERVICES: |
+      { 
+        "en": "https://www.sspcloud.fr/tos_en.md", 
+        "fr": "https://www.sspcloud.fr/tos_fr.md" 
+      }
 api:
   env:
     security.cors.allowed_origins: "http://localhost:3000"
+    authentication.mode: openidconnect
+    keycloak.realm: datalab
+    keycloak.auth-server-url: https://auth.lab.my-domain.net/auth
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
              "authenticationMode":"admin",
              "expose":{
                 "domain":"lab.my-domain.net"
              },
              "monitoring":{
                 "URLPattern":"todo"
              },
              "cloudshell":{
                 "catalogId":"inseefrlab-helm-charts-datascience",
                 "packageName":"cloudshell"
              },
              "initScript":"https://inseefrlab.github.io/onyxia/onyxia-init.sh"
           },
           "data":{
              "S3":{
                 "URL":"todo",
                 "monitoring":{
                    "URLPattern":"todo"
                 }
              }
           },
           "auth":{
              "type":"openidconnect"
           },
           "location":{
              "lat":48.8164,
              "long":2.3174,
              "name":"Montrouge (France)"
           }
        }
     ]
```

Now that you have updated `onyxia-values.yaml` restart onyxia-web with the new configuration.

```bash
helm upgrade onyxia inseefrlab/onyxia -f onyxia-values.yaml
```

Now your users should be able to create account, log-in, and start services on their own Kubernetes namespace.

### S3 Storage

Onyxia-web use [AWS Security Token Service API](https://docs.aws.amazon.com/STS/latest/APIReference/welcome.html) to get token and empowered user with storage features. We support any S3 storage compatible with this API. In this context, we are using [MinIO](https://min.io/), which is compatible with the Amazon S3 storage service and we demonstrate how to integrate it with Keycloak.

#### Create a Keycloak client for Accessing Keycloak

Before configuring MinIO, let's create a new client for Keycloak (from the previous existing "datalab" realm).

Create a client called "minio".

1. _Client ID_: **minio**
2. _Client Protocol_: **openid-connect**
3. _Root URL_: **https://minio.lab.my-domain.net/**

Complete the content of client "minio" with the following values.

1. _Access Type_: **confidential**
2. _Valid Redirect URIs_ (two values are required): **https://minio.lab.my-domain.net/\*** and **https://minio-console.lab.my-domain.net/\***
3. _Web origins_: **\***

Save the content, a new tab called Credentials must be appear. Navigate to Credentials tab and copy the secret value for the next section.

Navigate to Mappers tab and create a protocol Mapper.

1. _Name_: **policy**
2. _Mapper Type_: **Hardcoded claim**

Complete the content of Mapper "policy" with the following values.

1. _Token Claim Name_: **policy**
2. _Claim value_: **stsonly**
3. _Add to ID token_: **on**
4. _Add to access token_: **on**
5. _Add to userinfo_: **on**

#### Install MinIO

We recommand you to follow [MinIO documentation](https://min.io/docs/minio/linux/administration/identity-access-management/oidc-access-management.html#minio-external-identity-management-openid) for this installation and you must activate OIDC authentification. We will use the official Helm in this tutorial. All Helm configuration values can be found within this [link](https://github.com/minio/minio/blob/master/helm/minio/values.yaml).

> Replace `COPY_SECRET_FROM_KEYCLOAK_MINIO_CLIENT` by the secret value defined into the "minio" Keycloak client (see previous section).

```bash
helm repo add minio https://charts.min.io/
 
DOMAIN=my-domain.net

cat << EOF > ./minio-values.yaml
## replicas: 16
ingress:
  enabled: true
  annotations:
    kubernetes.io/ingress.class: nginx
  path: /
  hosts:
    - minio.lab.$DOMAIN
  tls:
    - hosts:
        - minio.lab.$DOMAIN
consoleIngress:
  enabled: true
  annotations:
    kubernetes.io/ingress.class: nginx
  paths: /
  hosts:
    - minio-console.lab.$DOMAIN
  tls:
    - hosts:
        - minio-console.lab.$DOMAIN
environment:
  MINIO_BROWSER_REDIRECT_URL: https://minio-console.lab.$DOMAIN
oidc:
  enabled: true
  configUrl: "https://auth.lab.$DOMAIN/auth/realms/datalab/.well-known/openid-configuration"
  clientId: "minio"
  claimName: "policy"
  scopes: "openid,profile,email"
  redirectUri: "https://minio-console.lab.$DOMAIN/oauth_callback"
  claimPrefix: ""
  comment: ""
  clientSecret: COPY_SECRET_FROM_KEYCLOAK_MINIO_CLIENT
policies:
  - name: stsonly
    statements:
      - resources:
          - 'arn:aws:s3:::oidc-${jwt:preferred_username}'
          - 'arn:aws:s3:::oidc-${jwt:preferred_username}/*'
        actions:
          - "s3:*"
EOF

helm install minio minio/minio -f minio-values.yaml
```

MinIO is now deployed and is accessible on the console url.

> By default, there are 16 MinIO containers running. If this number is too large for your Kubernetes cluster, you can limit it by configuring the 'replicas' key.

#### Create a Keycloak Client for Onyxia/MinIO

Before configuring the onyxia region to create tokens we should go back to Keycloak and create a new client to enable onyxia-web to request token for MinIO. This client is a little bit more complexe than other if you want to manage durations (here 7 days) and this client should have a claim name policy and with a value of stsonly according to our last deployment of MinIO.

From "datalab" realm, create a client called "onyxia-minio"

1. _Client ID_: **onyxia-minio**
2. _Client Protocol_: **openid-connect**
3. _Root URL_: **https://onyxia.my-domain.net/**

Complete the content of client "onyxia-minio" with the following values.

1. _Access Type_: **public**
2. _Valid Redirect URIs_: **https://onyxia.my-domain.net/\***
3. _Web origins_: **\***
4. Advanced Settings\
   1\. Access Token Lifespan : 7 days\
   2\. Client Session Idle : 7 days\
   3\. Client Session Max: 7 days

Save the content and navigate to Mappers tab and create two protocol Mappers.

Create the first Mapper called "policy".

1. _Token Name_: **policy**
2. _Mapper Type_: **Hardcoded claim**
3. _Token Claim Name_: **policy**
4. _Claim value_: **stsonly**
5. _Add to ID token_: **on**
6. _Add to access token_: **on**
7. _Add to userinfo_: **on**

Create the second Mapper called "audience-minio".

1. _Token Name_: **audience-minio**
2. _Mapper Type_: **Audience**
3. \_Included Custom Audience \_: **minio**
4. _Add to ID token_: **on**
5. _Add to access token_: **on**

#### Update Onyxia

S3 storage is configured inside a region in Onyxia api. You have some options to configure this storage and let inform Onyxia web all needed informations how to generate those tokens : keycloak parameters to access storage API, duration of STS tokens, bucket name with a standard prefix and a claim in the user JWT token to generate a unique identifiant for this bucket name, whether Onyxia-web should try to to create this bucket silently or not. There is also options for projects. You should look all options for the version of your need on [github](https://github.com/InseeFrLab/onyxia-api/blob/master/docs/region-configuration.md#s3)

<pre class="language-diff"><code class="lang-diff">serviceAccount:
  clusterAdmin: true
 ingress:
   enabled: true
   annotations:
     kubernetes.io/ingress.class: nginx
   hosts:
     - host: onyxia.my-domain.net
 web:
  env:
    KEYCLOAK_REALM: datalab
    KEYCLOAK_URL: https://auth.lab.my-domain.net/auth
    TERMS_OF_SERVICES: |
      { "en": "https://www.sspcloud.fr/tos_en.md", "fr": "https://www.sspcloud.fr/tos_fr.md" }
 api:
   env:
     security.cors.allowed_origins: "http://localhost:3000"
    authentication.mode: openidconnect
    keycloak.realm: datalab
    keycloak.auth-server-url: https://auth.lab.my-domain.net/auth
   regions:
     [
        {
           "id":"demo",
           "name":"Demo",
           "description":"This is a demo region, feel free to try Onyxia !",
           "services":{
              "type":"KUBERNETES",
              "singleNamespace": false,
              "namespacePrefix":"user-",
              "usernamePrefix":"oidc-",
              "groupNamespacePrefix":"projet-",
              "groupPrefix":"oidc-",
              "authenticationMode":"admin",
              "expose":{
                 "domain":"lab.my-domain.net"
              },
              "monitoring":{
                 "URLPattern":"todo"
              },
              "cloudshell":{
                 "catalogId":"inseefrlab-helm-charts-datascience",
                 "packageName":"cloudshell"
              },
              "initScript":"https://inseefrlab.github.io/onyxia/onyxia-init.sh"
           },
           "data":{
              "S3":{
<strong>-                "URL":"todo",
</strong>+                "type": "minio",
+                "URL": "https://minio.lab.my-domain.net",
+                "region": "us-east-1",
+                "bucketPrefix": "oidc-",
+                "groupBucketPrefix": "projet-",
+                "bucketClaim": "preferred_username",
+                "defaultDurationSeconds": 86400,
+                "keycloakParams":
+                {
+                      "URL": "https://auth.lab.my-domain.net/auth",
+                      "realm": "datalab",
+                      "clientId": "onyxia-minio",
+                },
+                "acceptBucketCreation": true,
                 "monitoring":{
                    "URLPattern":"minio"
                 }
              }
           },
           "auth":{
              "type":"openidconnect"
           },
           "location":{
              "lat":48.8164,
              "long":2.3174,
              "name":"Montrouge (France)"
           }
        }
     ]
</code></pre>

```bash
helm upgrade onyxia inseefrlab/onyxia -f onyxia-values.yaml
```

### Vault

Onyxia-web use vault as a storage for two kinds of secrets :\
1\. secrets or information generate by Onyxia to store differents values (ui preferences for example)\
2\. user secrets\
\
Vault must be configured with JWT or OIDC authentification methods.

As vault need to be initialized with a master key, It can't be directly configured with all parameters such as oidc or access policies and roles. So first step we create a vault with dev mode (do not use this in production and do your initialization with any of the recommanded configuration : shamir, gcp, another vault)

```bash
helm repo add hashicorp https://helm.releases.hashicorp.com
 
DOMAIN=my-domain.net

cat << EOF > ./vault-values.yaml
server:
  dev:
    enabled: true
    # Set VAULT_DEV_ROOT_TOKEN_ID value
    devRootToken: "root"
  ingress:
    enabled: true
    annotations:
      kubernetes.io/ingress.class: nginx
    hosts:
      - host: "vault.lab.$DOMAIN"
    tls:
      - hosts:
          - vault.lab.$DOMAIN
EOF

helm install vault hashicorp/vault -f vault-values.yaml
```

Create a client called "vault"

1. _Root URL_: **https://vault.lab.my-domain.net/**
2. _Valid redirect URIs_: **https://vault.lab.my-domain.net/\***
3. _Web origins_: **\***

TODO; [Refer to the legacy documentation.](https://github.com/InseeFrLab/onyxia/tree/main/step-by-step#set-up-authentication-openidconnect)
