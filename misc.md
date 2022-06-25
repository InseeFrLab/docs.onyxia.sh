# 💭 Misc

### Test your Kubernetes cluster

Make sure that everything is working fine on your cluster by deploying a simple app. &#x20;

![The hello world SPA deployed](<.gitbook/assets/image (8).png>)

```bash
cat << EOF > ./values.yaml
ingress:
  enabled: true
  annotations:
    kubernetes.io/ingress.class: nginx
  hosts:
    - host: my-test-spa.lab.my-domain.net
image:
  version: "0.4.3"
EOF

helm repo add etalab https://etalab.github.io/helm-charts
helm install my-test-spa etalab/keycloakify-demo-app -f values.yaml

#Navigate to https://my-test-spa.lab.my-domain.net, see the Hello World

helm uninstall my-test-spa
```
