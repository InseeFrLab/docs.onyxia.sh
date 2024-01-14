---
description: Provision a Kubernetes cluster
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

# 🛳 Kubernetes

First you'll need a Kubernetes cluster. If you have one already you can skip and directly go to [the Onyxia instalation section](gitops.md).

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
datalab.my-domain.net CNAME xxx.elb.eu-west-1.amazonaws.com. 
*.lab.my-domain.net  CNAME xxx.elb.eu-west-1.amazonaws.com. 
```

If the address you got was an IPv4 (`x.x.x.x`), create a `A` record instead of a CNAME.

If the address you got was ans IPv6 (`y:y:y:y:y:y:y:y`), create a `AAAA` record.

**https://datalab.my-domain.net** will be the URL for your instance of Onyxia. The URL of the services created by Onyxia are going to look like: **https://\<something>.lab.my-domain.net**

{% hint style="info" %}
You can customise "**datalab**" and "**lab**" to your liking, for example you could chose **onyxia.my-domain.net** and **\*.kub.my-domain.net**.
{% endhint %}

**SSL**

In this section we will obtain a TLS certificate issued by [LetsEncrypt](https://letsencrypt.org/) using the [certbot](https://certbot.eff.org/) commend line tool then get our ingress controller to use it.

If you are already familiar with `certbot` you're probably used to run it on a remote host via SSH. In this case you are expected to run it on your own machine, we'll use the DNS chalenge instead of the HTTP chalenge.

```bash
brew install certbot #On Mac, lookup how to install certbot for your OS

#Because we need a wildcard certificate we have to complete the DNS callange.  
sudo certbot certonly --manual --preferred-challenges dns

# When asked for the domains you wish to optains a certificate for enter:
#   datalab.my-domain.net *.lab.my-domain.net
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
    --key /etc/letsencrypt/live/datalab.$DOMAIN/privkey.pem \
    --cert /etc/letsencrypt/live/datalab.$DOMAIN/fullchain.pem
```

Lastly, we want to tell our ingress controller to use this TLS certificate, to do so run:

```bash
kubectl edit deployment ingress-nginx-controller -n ingress-nginx
```

This command will open your configured text editor, go to line `56` and add:

```
      - --default-ssl-certificate=ingress-nginx/onyxia-tls
```

![](<../../.gitbook/assets/image (13).png>)
{% endtab %}

{% tab title="Test on your machine" %}
If you are on a Mac or Window computer you can install [Docker desktop](https://www.docker.com/products/docker-desktop/) then enable Kubernetes.

<figure><img src="../../.gitbook/assets/image (34).png" alt=""><figcaption><p>Enabling Kubernetes in the Docker desktop App</p></figcaption></figure>

{% hint style="info" %}
Docker desktop isn't available on Linux, you can use [Kind](https://kind.sigs.k8s.io/) instead.
{% endhint %}

**Port Forwarding**

You'll need to [forward the TCP ports 80 and 443 to your local machine](https://user-images.githubusercontent.com/6702424/174459930-23fb577c-11a2-49ef-a082-873f4139aca1.png). It's done from the administration panel of your domestic internet Box. If you're on a corporate network you'll have to [test onyxia on a remote Kubernetes cluster](kubernetes.md#provisioning-a-cluster-on-aws-gcp-or-azure).

**DNS**

Let's assume you own the domain name **my-domain.net,** for the rest of the guide you should replace **my-domain.net** by a domain you actually own.

Get [your internet box routable IP](http://monip.org/) and create the following DNS records:

```dns-zone-file
datalab.my-domain.net A <YOUR_IP>
*.lab.my-domain.net   A <YOUR_IP>
```

{% hint style="success" %}
If you have DDNS domain you can create `CNAME` instead example:

```
datalab.my-domain.net CNAME jhon-doe-home.ddns.net.
*.lab.my-domain.net   CNAME jhon-doe-home.ddnc.net.
```
{% endhint %}

_**https://onyxia.my-domain.net**_ will be the URL for your instance of Onyxia.

The URL of the services created by Onyxia are going to look like: _**https://xxx.lab.my-domain.net**_

{% hint style="info" %}
You can customise "**datalab**" and "**lab**" to your liking, for example you could chose **onyxia.my-domain.net** and **\*.kub.my-domain.net**.
{% endhint %}

**SSL**

In this section we will obtain a TLS certificate issued by [LetsEncrypt](https://letsencrypt.org/) using the [certbot](https://certbot.eff.org/) commend line tool.

```bash
brew install certbot #On Mac, lookup how to install certbot for your OS

# Because we need a wildcard certificate we have to complete the DNS callange.  
sudo certbot certonly --manual --preferred-challenges dns

# When asked for the domains you wish to optains a certificate for enter:
#   datalab.my-domain.net *.lab.my-domain.net
```

{% hint style="info" %}
The obtained certificate needs to be renewed every three month.

To avoid the burden of having to remember to re-run the `certbot` command periodically you can setup [cert-manager](https://cert-manager.io/) and configure a [DNS01 challenge provider](https://cert-manager.io/docs/configuration/acme/dns01/) on your cluster but that's out of scope for Onyxia.

You may need to delegate your DNS Servers to one of the supported [DNS service provider](https://cert-manager.io/docs/configuration/acme/dns01/#supported-dns01-providers).
{% endhint %}

Now we want to create a Kubernetes secret containing our newly obtained certificate:

```bash
# First let's make sure we connect to our local Kube cluser
kubectl config use-context docker-desktop

kubectl create namespace ingress-nginx
DOMAIN=my-domain.net
sudo kubectl create secret tls onyxia-tls \
    -n ingress-nginx \
    --key /etc/letsencrypt/live/datalab.$DOMAIN/privkey.pem \
    --cert /etc/letsencrypt/live/datalab.$DOMAIN/fullchain.pem
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

Now that we have a Kubernetes cluster  ready to use let's levrage ArgoCD and GitOps practices to deploy and monitor the core services of our Onyxia Datalab. &#x20;

{% content-ref url="gitops.md" %}
[gitops.md](gitops.md)
{% endcontent-ref %}
