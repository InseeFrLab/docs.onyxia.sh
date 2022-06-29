# 🏁 Installing

This is a step by step guide that will assist you in installing Onyxia. &#x20;

### Provision a Kubernetes cluster

First you'll need a Kubernetes cluster.  If you have one already you can skip this section.

{% tabs %}
{% tab title="Provisioning a cluster on AWS, GCP or Azure" %}
[Hashicorp](https://www.hashicorp.com/) maintains great tutorials for [terraforming](https://www.terraform.io/) Kubernetes clusters on [AWS](https://aws.amazon.com/what-is-aws/), [GCP](https://cloud.google.com/) or [Azure](https://acloudguru.com/videos/acg-fundamentals/what-is-microsoft-azure).&#x20;

Pick one of the three and follow the guide.&#x20;

You can stop after the [configure kubectl section](https://learn.hashicorp.com/tutorials/terraform/eks#configure-kubectl). &#x20;

{% embed url="https://learn.hashicorp.com/tutorials/terraform/eks" %}

{% embed url="https://learn.hashicorp.com/tutorials/terraform/gke?in=terraform/kubernetes" %}

{% embed url="https://learn.hashicorp.com/tutorials/terraform/aks?in=terraform/kubernetes" %}

### Ingress controller &#x20;

Deploy an ingress controller on your cluster:

```bash
# AWS: 
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.2.0/deploy/static/provider/aws/deploy.yaml
```

{% hint style="warning" %}
[The above command](https://kubernetes.github.io/ingress-nginx/deploy/#aws) is for AWS. &#x20;

For GCP use [this command](https://kubernetes.github.io/ingress-nginx/deploy/#gce-gke).

For Azure use [this command](https://kubernetes.github.io/ingress-nginx/deploy/#azure).
{% endhint %}

### DNS

Let's assume you own the domain name **my-domain.net**, for the rest of the guide you should replace **my-domain.net** by a domain you actually own. &#x20;

Now you need to get the external address of your cluster, run the command&#x20;

```bash
kubectl get services -n ingress-nginx
```

and write down the `External IP` assigned to the `LoadBalancer`.&#x20;

Depending on the cloud provider you are using it can be an IPv4, an IPv6 or a domain. On AWS for example, it will be a domain like **xxx.elb.eu-west-1.amazonaws.com**. &#x20;

If you see `<pending>`, wait a few seconds and try again. &#x20;

Once you have the address, create the following DNS records:

```dns-zone-file
onyxia.my-domain.net CNAME xxx.elb.eu-west-1.amazonaws.com. 
*.lab.my-domain.net  CNAME xxx.elb.eu-west-1.amazonaws.com. 
```

If the address you got was an IPv4 (`x.x.x.x`), create a `A` record instead of a CNAME.

If the address you got was ans IPv6 (`y:y:y:y:y:y:y:y`), create a `AAAA` record.

**https://onyxia.my-domain.net** will be the URL for your instance of Onyxia. The URL of the services created by Onyxia are going to look like: **https://\<something>.lab.my-domain.net**&#x20;

{% hint style="info" %}
You can customise "**onyxia**" and "**lab**" to your liking, for example you could chose **datalab.my-domain.net** and **\*.kub.my-domain.net**.
{% endhint %}

### SSL

In this section we will obtain a TLS certificate issued by [LetsEncrypt](https://letsencrypt.org/) using the [certbot](https://certbot.eff.org/) commend line tool then get our ingress controller to use it. &#x20;

```bash
brew install certbot #On Mac, lookup how to install certbot for your OS

#Because we need a wildcard certificate we have to complete the DNS callange.  
sudo certbot certonly --manual --preferred-challenges dns

# When asked for the domains you wish to optains a certificate for enter:
#   *.lab.my-domain.net onyxia.my-domain.net
```

{% hint style="info" %}
The obtained certificate needs to be renewed every three month. &#x20;

To avoid the burden of having to remember to re-run the `certbot` command periodically you can setup [cert-manager](https://cert-manager.io/) and configure a [DNS01 provider](https://cert-manager.io/docs/configuration/acme/dns01/#delegated-domains-for-dns01) on your cluster but that's out of scope for Onyxia.
{% endhint %}

Now we want to create a Kubernetes secret containing our newly obtained certificate: &#x20;

```bash
DOMAIN=my-domain.net
sudo kubectl create secret tls onyxia-tls \
    -n ingress-nginx \
    --key /etc/letsencrypt/live/lab.$DOMAIN/privkey.pem \
    --cert /etc/letsencrypt/live/lab.$DOMAIN/fullchain.pem
```

Lastly, we want to tell our ingress controller to use this TLS certificate, to do so run: &#x20;

```bash
kubectl edit deployment ingress-nginx-controller -n ingress-nginx
```

This command will open your configured text editor, go to line `56` and add: &#x20;

```yaml
        - --default-ssl-certificate=ingress-nginx/onyxia-tls
```

![](<.gitbook/assets/image (1).png>)
{% endtab %}

{% tab title="Test on your machine" %}
If you are on a Mac or Window computer you can install [Docker desktop](https://www.docker.com/products/docker-desktop/) then enable Kubernetes.

![Enable Kubernetes in Docker desktop](<.gitbook/assets/image (5).png>)

{% hint style="info" %}
Docker desktop isn't available on Linux, you can use [Kind](https://kind.sigs.k8s.io/) instead.
{% endhint %}

### Port Forwarding

You'll need to [forward the TCP ports 80 and 443 to your local machine](https://user-images.githubusercontent.com/6702424/174459930-23fb577c-11a2-49ef-a082-873f4139aca1.png).  It's done from the administration panel of your domestic internet Box. If you're on a corporate network, no luck for you I'm afraid.

### DNS

Let's assume you own the domain name **my-domain.net,** for the rest of the guide you should replace **my-domain.net** by a domain you actually own.

Get [your internet box routable IP](http://monip.org/) and create the following DNS records: &#x20;

```dns-zone-file
onyxia.my-domain.net A <YOUR_IP>
*.lab.my-domain.net  A <YOUR_IP>
```

{% hint style="success" %}
If you have DDNS domain you can create `CNAME` instead example: &#x20;

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

### SSL

In this section we will obtain a TLS certificate issued by [LetsEncrypt](https://letsencrypt.org/) using the [certbot](https://certbot.eff.org/) commend line tool. &#x20;

```bash
brew install certbot #On Mac, lookup how to install certbot for your OS

#Because we need a wildcard certificate we have to complete the DNS callange.  
sudo certbot certonly --manual --preferred-challenges dns

# When asked for the domains you wish to optains a certificate for enter:
#   *.lab.my-domain.net onyxia.my-domain.net 
```

{% hint style="info" %}
The obtained certificate needs to be renewed every three month. &#x20;

To avoid the burden of having to remember to re-run the `certbot` command periodically you can setup [cert-manager](https://cert-manager.io/) and configure a [DNS01 provider](https://cert-manager.io/docs/configuration/acme/dns01/#delegated-domains-for-dns01) on your cluster but that's out of scope for Onyxia. &#x20;
{% endhint %}

Now we want to create a Kubernetes secret containing our newly obtained certificate: &#x20;

```bash
kubectl create namespace ingress-nginx
DOMAIN=my-domain.net
sudo kubectl create secret tls onyxia-tls \
    -n ingress-nginx \
    --key /etc/letsencrypt/live/lab.$DOMAIN/privkey.pem \
    --cert /etc/letsencrypt/live/lab.$DOMAIN/fullchain.pem
```

### Ingress controller

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
    -f ./ingress-nginx-values.yamlIf you want to confirm that you are ready to move over to the next step you can .
```
{% endtab %}
{% endtabs %}

### Installing Onyxia using helm

In this section we assume that:&#x20;

* You have a Kubernetes cluster and `kubectl` configured
* **onyxia.my-domain.net** and **\*.lab.my-domain.net** are pointing to your cluster's external address. **my-domain.net** being a domain that you own. You can customise "**onyxia**" and "**lab**" to your liking, for example you could chose **datalab.my-domain.net** and **\*.kub.my-domain.net**.
* You have an ingress controller configured with a default TLS certificate for **\*.lab.my-domain.net** and **onyxia.my-domain.net**. &#x20;

{% hint style="warning" %}
As of today [the default service catalog](https://github.com/InseeFrLab/helm-charts-datascience) will only work with [ingress-nginx](https://kubernetes.github.io/ingress-nginx/). &#x20;

This will be addressed in the near future. &#x20;
{% endhint %}

<details>

<summary>(Optional) Make sure that your cluster is ready for Onyxia</summary>

To make sure that your Kubernetes cluster is correctly configured let's deploy a test web app on it before deploying Onyxia.  &#x20;

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
image:
  version: "0.4.3"
EOF

helm repo add etalab https://etalab.github.io/helm-charts
helm install test-spa etalab/keycloakify-demo-app -f test-spa-values.yaml
echo "Navigate to https://test-spa.lab.$DOMAIN, see the Hello World"
helm uninstall test-spa
```

</details>

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
ui:
  image:
    # Update on your own therm but update!
    # https://hub.docker.com/r/inseefrlab/onyxia-api/tags
    version: "0.56.3"
api:
  image:
    # Same here
    # https://hub.docker.com/r/inseefrlab/onyxia-api/tags
    version: "v0.11"
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

helm install onyxia inseefrlab/onyxia -f onyxia-values.yaml
```

&#x20;You can now access `https://onyxia.my-domain.net` and start services. Congratulations! 🥳

At the moment the Onyxia you just deployed is running in degraded mode, there is no user authentication and no s3 integration for example. &#x20;

[Let's see how to enable theres features.](https://github.com/InseeFrLab/onyxia/tree/main/step-by-step#set-up-authentication-openidconnect)

### Enabling user authentication

At the moment there is no authentication process, everyone can access our platform and and start services on the default namespace. &#x20;

Let's setup Keycloak to enable users to create account and login to our Onyxia. &#x20;

We will also see how to restrict registration to our service with a email domain accept list. &#x20;

For deploying our Keycloak we use [codecentric's helm chart](https://github.com/codecentric/helm-charts/tree/master/charts/keycloak). &#x20;

```bash
cat << EOF > ./keycloak-values.yaml
image:
  tag: "18.0.0-legacy"
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
        curl -L -f -S -o /extensions/onyxia-web.jar https://github.com/InseeFrLab/onyxia-web/releases/latest/download/keycloak-theme.jar
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
      -Dkeycloak.profile=preview -XX:+UseContainerSupport -XX:MaxRAMPercentage=50.0 -Djava.net.preferIPv4Stack=true -Djboss.modules.system.pkgs=\$JBOSS_MODULES_SYSTEM_PKGS -Djava.awt.headless=true 
ingress:
  enabled: true
  servicePort: http
  annotations:
    ## Resolve HTTP 502 error using ingress-nginx:
    ## See https://www.ibm.com/support/pages/502-error-ingress-keycloak-response
    nginx.ingress.kubernetes.io/proxy-buffer-size: 128k
  rules:
    - host: "sill-auth.etalab.gouv.fr"
      paths:
        - path: /
          pathType: Prefix
  tls:
    - hosts:
        - sill-auth.etalab.gouv.fr
      secretName: sill-tls
postgresql:
  postgresqlPassword: $POSTGRESQL_PASSWORD
EOF
```



"Root URL": "https://onyxia.my-domain.net" and&#x20;

"Valid redirect URIs": "https://onyxia.my-domain.net/\*"

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

Update the `onyxia-values.yaml` file  that you created previously, don't forget to replace all the occurence of **my-domain.net** by your actual domain.

```diff
helm upgrade onyxia inseefrlab/onyxia -f onyxia-values.yaml
```

