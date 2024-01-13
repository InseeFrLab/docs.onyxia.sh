---
description: Convinced by Onyxia? Let's see how you can get your own instance today!
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

{% hint style="info" %}
## Oneliner

If you are already familiar with Kubernetes and Helm, here's how you can get an Onyxia instance up and running in just a matter of seconds.

```bash
helm repo add onyxia https://inseefrlab.github.io/onyxia

cat << EOF > ./onyxia-values.yaml
ingress:
  enabled: true
  hosts:
    - host: onyxia.my-domain.net
EOF

helm install onyxia onyxia/onyxia -f onyxia-values.yaml

# Navigate to https://onyxia.my-domain.net
```

With this minimal configuration, you'll have an Onyxia instance operating in a degraded mode, which lacks features such as authentication, S3 explorer, secret management, etc. However, you will still retain the capability to launch services from the catalog.
{% endhint %}

Whether you are a Kubernetes veteran or a beginner with cloud technologies, this guide aims to guide you through the instantiation and configuration of an Onyxia instance. Let's dive right in!

First let's make sure we have a suitable deployment environement to work with: &#x20;

{% content-ref url="readme/kubernetes.md" %}
[kubernetes.md](readme/kubernetes.md)
{% endcontent-ref %}
