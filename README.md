# 🏁 Installing

This is a step by step guide that will assist you in installing Onyxia. &#x20;

### Provision a Kubernetes cluster

First you'll need a Kubernetes (k8s) cluster.  If you have one already you can skip this section.

{% tabs %}
{% tab title="Test locally" %}
If you are on a Mac or Window computer you can install Docker desktop then enable Kubernetes.

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
`"lab"` and `"onyxia"` are mere suggestions.
{% endhint %}

### SSL

In this section we will obtain a TLS certificate issued by [LetsEncrypt](https://letsencrypt.org/) using the [certbot](https://certbot.eff.org/) commend line tool. &#x20;

```bash
brew install certbot #On Mac, lookup how to install certbot for your OS

#Because we need a wildcard we have to complete the DNS callange.  
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
sudo kubectl create secret tls onyxia-tls \
    -n ingress-nginx \
    --key /etc/letsencrypt/live/lab.my-domain.net/privkey.pem \
    --cert /etc/letsencrypt/live/lab.my-domain.net/fullchain.pem
```

### Ingress controller

We'll install [ingress-nginx](https://kubernetes.github.io/ingress-nginx/) in our cluster but any other ingress controller will do.

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

{% tab title="Provisioning a cluster on AWS" %}
[Hashicorp](https://www.hashicorp.com/) maintains a great tutorial for getting started with [terraform](https://www.terraform.io/) on [AWS](https://aws.amazon.com/free/?trk=7214f2bf-dcfb-4d46-8a27-608345ad6b51\&sc\_channel=ps\&sc\_campaign=acquisition\&sc\_medium=ACQ-P|PS-GO|Brand|Desktop|SU|Core-Main|Core|FR|EN|Text\&s\_kwcid=AL!4422!3!454820903991!e!!g!!amazon%20web%20services\&ef\_id=CjwKCAjw77WVBhBuEiwAJ-YoJLQXmQRATR7nW2rfWnU3Rk704sb4-ggXveYG47rwLNq\_wBgX8SkNNRoCLy0QAvD\_BwE:G:s\&s\_kwcid=AL!4422!3!454820903991!e!!g!!amazon%20web%20services\&all-free-tier.sort-by=item.additionalFields.SortRank\&all-free-tier.sort-order=asc\&awsf.Free%20Tier%20Types=\*all\&awsf.Free%20Tier%20Categories=\*all) and deploying a basic Kubernetes cluster:

{% embed url="https://learn.hashicorp.com/tutorials/terraform/eks" %}

You can stop after the [configure kubectl section](https://learn.hashicorp.com/tutorials/terraform/eks#configure-kubectl). &#x20;

### Ingress controller &#x20;

Deploy an ingress controller on your cluster:

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.2.0/deploy/static/provider/aws/deploy.yaml
```

(This commend was taken from [this guide](https://kubernetes.github.io/ingress-nginx/deploy/#aws))

### DNS

Let's assume you own the domain name **my-domain.net**, for the rest of the guide you should replace **my-domain.net** by a domain you actually own. &#x20;

Now you need to get the external IP of your cluster: run the command&#x20;

```bash
kubectl get services -n ingress-nginx
```

and write down the External IP. It should be something like:&#x20;

**xxx.elb.eu-west-1.amazonaws.com.** &#x20;

```dns-zone-file
onyxia.my-domain.net CNAME xxx.elb.eu-west-1.amazonaws.com. 
*.lab.my-domain.net  CNAME xxx.elb.eu-west-1.amazonaws.com. 
```

**https://onyxia.my-domain.net** will be the URL for your instance of Onyxia. The URL of the services created by Onyxia are going to look like: **https://xxx.lab.my-domain.net**&#x20;

{% hint style="info" %}
"lab" and "onyxia" are mere suggestions &#x20;
{% endhint %}

### SSL

In this section we will obtain a TLS certificate issued by [LetsEncrypt](https://letsencrypt.org/) using the [certbot](https://certbot.eff.org/) commend line tool then get our ingress controller to use it. &#x20;

```bash
brew install certbot #On Mac, lookup how to install certbot for your OS

#Because we need a wildcard we have to complete the DNS callange.  
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
sudo kubectl create secret tls onyxia-tls \
    -n ingress-nginx \
    --key /etc/letsencrypt/live/lab.my-domain.net/privkey.pem \
    --cert /etc/letsencrypt/live/lab.my-domain.net/fullchain.pem
```

Lastly, we want to tell our ingress controller to use this TLS certificate, to do so run: &#x20;

```bash
kubectl edit deployment ingress-nginx-controller -n ingress-nginx
```

This command will open your configured text editor, go to line `56` and add: &#x20;

```yaml
        - --default-ssl-certificate=ingress-nginx/wildcard
```

![](<.gitbook/assets/image (1).png>)
{% endtab %}
{% endtabs %}

### Installing Onyxia using helm

In this section we assume that:&#x20;

* You have a Kubernetes cluster and kubectl configured
* **onyxia.my-domain.net** and **\*.lab.my-domain.net** are pointing to your cluster's external address. **my-domain.net** being a domain that you own. You can customise "**onyxia**" and "**lab**" to your liking, for example you could chose **datalab.my-domain.net** and **\*.kub.my-domain.net**.
*









