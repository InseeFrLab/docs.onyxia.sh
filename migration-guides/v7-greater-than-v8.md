# ⬆ v7->v8

{% hint style="success" %}
You can now have comments, trailing comas and single quotes in your region and catalog parameters! See [the PR](https://github.com/InseeFrLab/onyxia-api/pull/344).
{% endhint %}

In this release, the Onyxia S3 integration has been completely revamped!&#x20;

{% embed url="https://github.com/InseeFrLab/onyxia-api/blob/main/docs/region-configuration.md#s3" %}
The new S3 region parameter specification
{% endembed %}

This is the DIFF you have to apply to your Onyxia configuration assuming you have a typical MinIO integration configured:   &#x20;

{% code title="onyxia-values.yaml" %}
```diff
 onyxia:
   ...
   api:
     ...
     regions:
       [
         {
           ...
           "data": {
             "S3": {
-              "type": "minio",
               "URL": "https://minio.lab.my-domain.net",
               "region": "us-east-1",
-              "bucketClaim": "preferred_username",
-              "defaultDurationSeconds": 86400,
-              "oidcConfiguration": {
-                "clientID": "onyxia-minio"
-              },
+              "sts": {
+                "durationSeconds": 86400,
+                "oidcConfiguration": {
+                  "clientID": "onyxia-minio"
+                }
+              },
-              "bucketPrefix": "user-",
-              "groupBucketPrefix": "projet-",
+              "workingDirectory": {
+                "bucketMode": "multi",
+                "bucketNamePrefix": "user-",
+                "bucketNamePrefixGroup": "projet-"
+              }
             },
             ...

```
{% endcode %}

```bash
helm upgrade onyxia inseefrlab/onyxia -f onyxia-values.yaml
```
