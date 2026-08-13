---
icon: sign-posts-wrench
---

# S3 Explorer Standalone Deployment

Onyxia features a fully client side (the web app talks directly to the s3 server with no proxy in the middle) S3 explorer.  \
It lets you browse your S3 buckets with an UI that it akin to Google Drive or Dropbox. &#x20;

## Simple deployment

No authentication mode:

{% code title="" %}
```bash
helm repo add onyxia https://inseefrlab.github.io/onyxia

cat << EOF > ./onyxia-values.yaml
ingress:
  enabled: true
  hosts:
    - host: onyxia.my-domain.net
web:
  env:
    HEADER_TEXT_FOCUS: "S3 Explorer"
api:
  enabled: false
EOF

helm install onyxia onyxia/onyxia -f onyxia-values.yaml

# Navigate to https://onyxia.my-domain.net
```
{% endcode %}

In this mode, when you access the app (https://onyxia.my-domain.net) you will be asked to create a S3 profile: Give the URL of a S3 server, the region, and optionally the Credentials (Access Key ID, Secret Access Key). Then you'll be able to browse the buckets you have access to.  \
Note that this configuration is saved in the local storage of the browser.  \
\
Note however that, since this explorer is soleiy browser based, if you try to browse an arbitrary S3 bucket it will may fail unless CORS have been enabled for arbitrary browser origins on that bucket.

## Deployment with user Authentication

Where the S3 Explorer featured by Onyxia really shine is when enabling OpenID Connect authentication.  \
What Onyxia enables you to do as an administrator is to make it so that users have to login to Keycloa, Auth0 or EntraID for example, then they can get access to a dedicated bucket or subpath that is only available to them. \
This is possible if your S3 storage support AssumeRoleWithWebIdentity.  <br>

If you want a complete tutorial for deploying Onyxia S3 Explorer on Kube alongside a Keycloak and Minio you can follow this tutoriel:

{% content-ref url="../../" %}
[..](../../)
{% endcontent-ref %}

In the data (S3) section you'll have a specific instruction for the mode S3 standalone.\
\
For other S3 storage (other than Minio) and Auth server (other than keycloak) refer to these documentation pages:

{% content-ref url="../s3-configuration.md" %}
[s3-configuration.md](../s3-configuration.md)
{% endcontent-ref %}

{% content-ref url="../openid-connect-configuration.md" %}
[openid-connect-configuration.md](../openid-connect-configuration.md)
{% endcontent-ref %}

Important note for the S3 standalone mode:

When Onyxia is deployed as a datalab, the S3 configuration are passed to the api region. Like [here](https://github.com/InseeFrLab/onyxia-ops/blob/379526d6faa2df935e656427a8b4dd21128d9bf6/apps/onyxia/values-minio-enabled.yaml#L66-L90). But in standalone mode, the parameter are passed as env to the web component. As shown [here](https://github.com/InseeFrLab/onyxia-ops/blob/main/apps/onyxia/values-s3-explorer-only.yaml).&#x20;

