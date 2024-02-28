---
description: Enable S3 storage via MinIO S3
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

# 🗃️ Data (S3)

Onyxia uses [AWS Security Token Service API](https://docs.aws.amazon.com/STS/latest/APIReference/welcome.html) to optain S3 tokens on behaf of your users. We support any S3 storage compatible with this API. In this context, we are using [MinIO](https://min.io/), which is compatible with the Amazon S3 storage service and we demonstrate how to integrate it with Keycloak.

### Creating the 'minio' Keycloak client

Before configuring MinIO, let's create a new Keycloak client (from the previous existing "datalab" realm).

{% embed url="https://app.tango.us/app/embed/1c5c0975-93f0-48c6-b8d9-edceb397e34c" %}

### Deploying MinIO

Before deploying MinIO on the cluster let's set, in the MinIO configuration file, the OIDC client secret we have copied in the previous setp. &#x20;

```bash
git clone https://github.com/<your-github-org>/onyxia-ops
cd onyxia-ops
cd apps/minio
# In the values.yaml file replace `$KEYCLOAK_MINIO_CLIENT_SECRET` by the value
# you have copied in the previous step.
git commit -am "Set minio OIDC client secret"
git push
```

Once you've done that you can deploy MinIO! &#x20;

{% embed url="https://app.tango.us/app/embed/75b62573-7adc-4a38-b1f9-b96bb0ea50fd" %}

### Creatint the 'onyxia-minio' Keycloak client

Before configuring the onyxia region to create tokens we should go back to Keycloak and create a new client to enable onyxia-web to request token for MinIO. This client is a little bit more complexe than other if you want to manage durations (here 7 days) and this client should have a claim name policy and with a value of stsonly according to our last deployment of MinIO.

{% embed url="https://app.tango.us/app/embed/2e382be2-5d73-4cc8-8682-1b86b0e1de58" %}

### Updating the Onyxia configuration

Now let's update our Onyxia configuration to let it know that there is now a S3 server available on the cluster. &#x20;

```bash
git clone https://github.com/<your-github-org>/onyxia-ops
cd onyxia-ops
cd apps/onyxia
mv values-minio-enabled.yaml.yaml values.yaml
git commit -am "Enable MinIO"
git push
```

Diff of the changes applied to the Onyxia configuration: &#x20;

{% embed url="https://github.com/InseeFrLab/onyxia-ops/commit/e8e5d57743d9954f60213346e33b55d2de41f707" %}

Congratulation, all the S3 related features of Onyxia are now enabled in your instance! Now if you navigate to your Onyxia instance you should have `My Files` in the left menu. &#x20;

<figure><img src="../../.gitbook/assets/image (37).png" alt=""><figcaption></figcaption></figure>

Next step in the installation process is to setup Vault to provide a way to your user so store secret and also to provide something that Onyxia can use as a persistance layer for user configurations.

{% content-ref url="...rest.md" %}
[...rest.md](...rest.md)
{% endcontent-ref %}
