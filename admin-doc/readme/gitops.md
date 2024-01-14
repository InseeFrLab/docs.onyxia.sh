---
description: Let's install ArgoCD to manage and monitor our Onyxia Datalab deployment!
---

# 🐙 GitOps

{% hint style="info" %}
At this stage of this installation process we assumes that: &#x20;

* You have a Kubernetes cluster and `kubectl` configured
* **datalab.my-domain.net** and **\*.lab.my-domain.net**'sDNS are pointing to your cluster's external address. **my-domain.net** being a domain that you own**.**
* You have an ingress-ngnix configured with a default TLS certificate for **\*.lab.my-domain.net** and **datalab.my-domain.net**.   &#x20;
{% endhint %}

We can proceed with manually installing various services via Helm to set up the datalab. However, it's more convenient and reproducible to maintain a Git repository that outlines the required services that we need for our datalab, allowing [ArgoCD](https://argo-cd.readthedocs.io/en/stable/) to handle the deployment for us. &#x20;

Let's install ArgoCD on the our cluster. &#x20;

```bash
helm repo add argo https://argoproj.github.io/argo-helm

DOMAIN=my-domain.net

cat << EOF > ./argocd-values.yaml
server:
  extraArgs:
    - --insecure
  ingress:
    enabled: true
    annotations:
      kubernetes.io/ingress.class: nginx
      nginx.ingress.kubernetes.io/ssl-passthrough: "true"
    hosts:
      - argocd.lab.$DOMAIN
    tls:
      - hosts:
          - argocd.lab.$DOMAIN
EOF

helm install argocd argo/argo-cd -f argocd-values.yaml
```

Now you have to get the password that have been automatically generated to protect ArgoCD's admin console, running this command will print the password:

```bash
kubectl -n default get secret argocd-initial-admin-secret \
  -o jsonpath="{.data.password}" | base64 -d
```

You can now login to **https://argocd.lab.my-domain.net** using: &#x20;

* username: **admin**
* password: **\<the output of the previous command (without the `%` at the end)>**



### Setting up your GitOps GitHub Repository

{% embed url="https://app.tango.us/app/embed/316721db-a2d7-4cc2-9e66-0be86f979a28" %}

