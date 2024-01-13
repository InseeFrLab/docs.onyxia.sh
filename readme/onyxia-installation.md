---
description: Installing Onyxia's Helm package on your Kubernetes cluster
layout:
  title:
    visible: true
  description:
    visible: true
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: false
---

# 🐉 Onyxia installation

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

<img src="../.gitbook/assets/image (23).png" alt="" data-size="original">
{% endhint %}

<details>

<summary>(Optional) Make sure that your cluster is ready for Onyxia</summary>

To make sure that your Kubernetes cluster is correctly configured let's deploy a test web app on it before deploying Onyxia.

<img src="../.gitbook/assets/image (16).png" alt="The hello world SPA deployed" data-size="original">

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

{% hint style="success" %}
Note that your region configuration can include comments 💬! &#x20;
{% endhint %}

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
  regions: 
    [
      {
        "id":"demo",
        "name":"Demo",
        "description":"This is a demo region, feel free to try Onyxia !",
        "services":{
          "type":"KUBERNETES",
          "singleNamespace": true,
          "authenticationMode": "serviceAccount",
          "expose":{
            "domain":"lab.$DOMAIN"
          }
        }
      }
    ]
EOF

helm install onyxia onyxia/onyxia -f onyxia-values.yaml
```

You can now access `https://onyxia.my-domain.net` and start services. Congratulations! 🥳

At this stage of the the installation process, everyone can access our platform and and start services.

Now let's see how to enables user authentication to your Onyxia instance and enables users to have their individual Kubernetes namespaces! &#x20;

{% content-ref url="user-authentication.md" %}
[user-authentication.md](user-authentication.md)
{% endcontent-ref %}
