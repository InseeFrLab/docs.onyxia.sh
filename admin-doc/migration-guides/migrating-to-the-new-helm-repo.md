# ⬆️ Migrating to the new helm repo

Previously, the Helm chart of Onyxia was hosted on the inseefrlab/helm-charts repo and has now been moved to inseefrlab/onyxia.  \
\
As a result you would now install Onyxia like this: &#x20;

```diff
-helm repo add inseefrlab https://inseefrlab.github.io/helm-charts
+helm repo add onyxia https://inseefrlab.github.io/onyxia

-helm install onyxia inseefrlab/helm-charts
+helm install onyxia onyxia/onyxia
```

In the following we assume the current version of Onyxia is 4.1.4 but you are encorging to use the latest version instead. [See releases](https://github.com/InseeFrLab/onyxia/releases).

If you use ArgoCD for deploying onyxia: &#x20;

<pre class="language-diff" data-title="apps/onyxia/Chart.yaml"><code class="lang-diff"><strong> apiVersion: v2
</strong> name: onyxia
 version: 1.0.0
 dependencies:
   - name: onyxia
-    version: 4.1.0
+    version: 4.1.4
-    repository: https://inseefrlab.github.io/helm-charts/
+    repository: https://inseefrlab.github.io/onyxia/
</code></pre>

You no longer need to manually manage the version of [onyxia-web](https://hub.docker.com/r/inseefrlab/onyxia-web) and [onyxia-api](https://hub.docker.com/r/inseefrlab/onyxia-api), now, if you want to update Onyxia, you just update the chart version number. &#x20;

```diff
helm repo add onyxia https://inseefrlab.github.io/onyxia

DOMAIN=my-domain.net

cat << EOF > ./onyxia-values.yaml
# ...
web:
  image:
-   tag: 2.29.4
api:
  image:
-   tag: v0.32   
# ...
EOF

helm install onyxia onyxia/onyxia -f onyxia-values.yaml
```

For the Keycloak theme, the version is now synchronized with the Onyxia version. &#x20;

```diff
helm repo add codecentric https://codecentric.github.io/helm-charts

cat << EOF > ./keycloak-values.yaml
# ... See https://docs.onyxia.sh/#enabling-user-authentication
extraInitContainers: |
  - name: realm-ext-provider
    image: curlimages/curl
    imagePullPolicy: IfNotPresent
    command:
      - sh
    args:
      - -c
      - |
-       curl -L -f -S -o /extensions/onyxia.jar https://github.com/InseeFrLab/onyxia/releases/download/v2.29.4/keycloak-theme.jar
+       curl -L -f -S -o /extensions/onyxia.jar https://github.com/InseeFrLab/onyxia/releases/download/v4.1.4/keycloak-theme.jar
    volumeMounts:
      - name: extensions
        mountPath: /extensions
extraVolumeMounts: |
  - name: extensions
    mountPath: /opt/jboss/keycloak/standalone/deployments
extraVolumes: |
  - name: extensions
    emptyDir: {}
# ...
EOF

helm install keycloak codecentric/keycloak -f keycloak-values.yaml
```

Also note that, the theme will now appear as "onyxia" in the dropdown. Previously it was "onyxia-web"\


<figure><img src="../../.gitbook/assets/image (44).png" alt=""><figcaption></figcaption></figure>
