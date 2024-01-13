---
description: Your Onyxia instance, today
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

# 🏁 Install

## Oneliner

If you are already familiar with  Kubernetes and Helm. Here is how you can get an Onyxia instance running in a matter of seconds.

```bash
helm repo add onyxia https://inseefrlab.github.io/onyxia

cat << EOF > ./onyxia-values.yaml
ingress:
  enabled: true
  hosts:
    - host: datalab.my-domain.net
EOF

helm install onyxia onyxia/onyxia -f onyxia-values.yaml
```

With this minimal configuration yield and Onyxia instance operating in a degraded mode, which lacks features such as authentication, S3 explorer, secret management, etc. However, you will still have the capability to launch services from the catalog.

## Step by step installation guide

Let's see how to instanciate an Onyxia instance with it's full range of feature! &#x20;

First let's make sure we have a suitable deployment environement to work with: &#x20;

{% content-ref url="readme/kubernetes.md" %}
[kubernetes.md](readme/kubernetes.md)
{% endcontent-ref %}
