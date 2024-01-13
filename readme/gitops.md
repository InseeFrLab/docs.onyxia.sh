---
description: Let's install ArgoCD to manage and monitor your Onyxia Datalab deployment!
---

# 🐙 GitOps

{% hint style="info" %}
At this stage of this installation process we assumes that: &#x20;

* You have a Kubernetes cluster and `kubectl` configured
* **onyxia.my-domain.net** and **\*.lab.my-domain.net** are pointing to your cluster's external address. **my-domain.net** being a domain that you own. You can customise "**onyxia**" and "**lab**" to your liking, for example you could chose **datalab.my-domain.net** and **\*.kub.my-domain.net**.
* You have an ingress-ngnix configured with a default TLS certificate for **\*.lab.my-domain.net** and **onyxia.my-domain.net**. (We are still working on making the default catalogs works with other ingress controllers). &#x20;
{% endhint %}

Let's set up ArgoCD on your cluster. ArgoCD enables you to monitor what's operating in your cluster and declaratively outline what should be deployed, along with the deployment methods. This approach marks a significant advancement over the imperative execution of commands, as it enhances the portability and replicability of your deployments. &#x20;

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
