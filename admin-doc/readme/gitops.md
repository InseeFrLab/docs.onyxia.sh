---
description: Let's install ArgoCD to manage and monitor our Onyxia Datalab deployment!
---

# 🐙 GitOps

{% hint style="info" %}
At this stage of this installation process we assumes that: &#x20;

* You have a Kubernetes cluster and `kubectl` configured
* **datalab.my-domain.net** and **\*.lab.my-domain.net**'sDNS are pointing to your cluster's external address. **my-domain.net** being a domain that you own**.**
* You have an ingress-ngnix configured with a default TLS certificate for both **datalab.my-domain.net and \*.lab.my-domain.net**.   &#x20;
{% endhint %}

We can proceed with manually installing various services via Helm to set up the datalab. However, it's more convenient and reproducible to maintain a Git repository that outlines the required services that we need for our datalab, allowing [ArgoCD](https://argo-cd.readthedocs.io/en/stable/) to handle the deployment for us. &#x20;

Let's install ArgoCD on the our cluster. &#x20;

```bash
DOMAIN=my-domain.net

cat << EOF > ./argocd-values.yaml
server:
  extraArgs:
    - --insecure
  ingress:
    enabled: true
    annotations:
      kubernetes.io/ingress.class: "nginx"
      nginx.ingress.kubernetes.io/ssl-passthrough: "true"
    hosts:
      - argocd.lab.$DOMAIN
    tls:
      - hosts:
          - argocd.lab.$DOMAIN
EOF

helm install argocd argo-cd \
  --repo https://argoproj.github.io/argo-helm \
  -f ./argocd-values.yaml
```

Now you have to get the password that have been automatically generated to protect ArgoCD's admin console, running this command will print the password:

```bash
kubectl get secret argocd-initial-admin-secret \
  -o jsonpath="{.data.password}" | base64 -d
```

You can now login to **https://argocd.lab.my-domain.net** using: &#x20;

* username: **admin**
* password: **\<the output of the previous command (without the `%` at the end)>**

<figure><img src="../../.gitbook/assets/image (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Now that we have an ArgoCD we want to connect it to a Git repository that will describe what services we want to be running on our cluster. &#x20;

Let's fork the onyxia-ops GitHub repo and use it to deploy an Onyxia instance!\


{% embed url="https://app.tango.us/app/embed/55af08f3-43b0-4b5d-84b7-dfb75f6983c9" %}

At this point you should have a very bare bone Onyxia instance that you can use to launch services. &#x20;

What's great, is that now, if you want to update the configuration of your Onyxia instance you only have to commit the change to your GitOps repo, ArgoCD will takes charge of restarting the service for you with the new configuration.  \
To put that to the test try to modify your Onyxia configuration

{% code title="apps/onyxia/values.yaml" %}
```diff
 onyxia:
   ingress:
     enabled: true
     annotations:
       kubernetes.io/ingress.class: nginx
     hosts:
       - host: datalab.demo-domain.ovh
+  web:
+    env:
+      HEADER_TEXT_BOLD: My Organization
   api:
     regions: [...]
```
{% endcode %}

After a few seconds, if you reload **https://datalab.my-domain.net** you should witness the update!

<figure><img src="../../.gitbook/assets/image (3).png" alt="" width="375"><figcaption><p>"My Organization" no appears in the Header</p></figcaption></figure>

Next step is to see how to enable your user to authenticate themselvs to your datalab!

{% content-ref url="user-authentication.md" %}
[user-authentication.md](user-authentication.md)
{% endcontent-ref %}

