# ⬆️ v5 -> v6

The only breaking change in this release is the split of Onyxia service account into two separate service accounts : one for the API (which usually requires high permission to deploy services) and one for the WEB pod (qui usually should not have any permissions tied to it).\
Due to this change, the global `serviceAccount` values key was duplicated in both `web.serviceAccount` and `api.serviceAccount`.\
\
See:

{% embed url="https://github.com/InseeFrLab/onyxia/blob/v6.0.1/helm-chart/values.yaml#L77" %}

&#x20;and&#x20;

{% embed url="https://github.com/InseeFrLab/onyxia/blob/v6.0.1/helm-chart/values.yaml#L160" %}

Example of change :

{% code title="onyxia-values.yaml" %}
```diff
onyxia:
-  serviceAccount:
-    create: true
-    clusterAdmin: true
   api:
+    serviceAccount:
+      create: true
+      clusterAdmin: true
   web:
+    serviceAccount:
+      create: true 
```
{% endcode %}
