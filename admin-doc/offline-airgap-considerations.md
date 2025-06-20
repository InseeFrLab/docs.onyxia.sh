---
icon: plane-circle-xmark
---

# Offline / airgap considerations

Onyxia can be installed in constrained environments such as behind a proxy, offline or airgap.  \
This page aims at listing various things and configurations to have in mind when installing Onyxia in such environments. &#x20;

### Catalogs &#x20;

By default, Onyxia (Onyxia-API to be precise) is configured to use \[Inseefrlab Opensource catalogs straight from Github]\([https://github.com/InseeFrLab/onyxia-api/blob/main/onyxia-api/src/main/resources/catalogs.json](https://github.com/InseeFrLab/onyxia-api/blob/main/onyxia-api/src/main/resources/catalogs.json)).  \
This won't work if you don't have access to internet.  \
If behind a proxy, you can \[configure the proxy]\([https://github.com/InseeFrLab/onyxia-api/tree/main?tab=readme-ov-file#http-configuration](https://github.com/InseeFrLab/onyxia-api/tree/main?tab=readme-ov-file#http-configuration)) by using the corresponding API env variables.  \
\
You can configure your own catalogs by using the \`catalogs\` key from the Helm chart : [https://github.com/InseeFrLab/onyxia/blob/1b404b5f043fc23e8e54bea1b7b3e163739d4404/helm-chart/values.yaml#L153](https://github.com/InseeFrLab/onyxia/blob/1b404b5f043fc23e8e54bea1b7b3e163739d4404/helm-chart/values.yaml#L153)\
A catalog is a regular Helm charts repository, see \[here]\([https://docs.onyxia.sh/admin-doc/catalog-of-services](https://docs.onyxia.sh/admin-doc/catalog-of-services)) for more details on how to create your own catalog.  \
Note that Onyxia does not currently support OCI-based repositories, you need to have an \`index.yaml\` based repository. See \[here]\([https://github.com/InseeFrLab/onyxia-api/issues/547](https://github.com/InseeFrLab/onyxia-api/issues/547)) to track progress on this.

### Certificates

If you are using non-public (internal) certificates, you need to either mount them (recommended) or skip tls validation (not recommended). &#x20;

#### Mounting certificates (recommended)&#x20;

Certificates can be mounted on the API pod :

```
api:
  extraVolumeMounts:
    - mountPath: "/usr/local/share/ca-certificates"
      name: ca-bundle
  extraVolumes:
    - name: ca-bundle
      secret:
        secretName: ca-bundle
```

#### Disabling tls validation (not recommended)

To disable tls validation for the API ⇒ OIDC provider : \`oidc.skip-tls-verify\`  \
To disable tls validation for Helm (catalogs retrieval) : \[skipTlsVerify]\([https://github.com/InseeFrLab/onyxia-api/blob/b47eece8103fa6bc78302390b3f0b8570de9e494/onyxia-api/src/main/resources/catalogs.json#L20](https://github.com/InseeFrLab/onyxia-api/blob/b47eece8103fa6bc78302390b3f0b8570de9e494/onyxia-api/src/main/resources/catalogs.json#L20))

### Images

Currently, Onyxia's images and images used by our opensource catalogs are hosted on \[Dockerhub]\([https://hub.docker.com/u/inseefrlab](https://hub.docker.com/u/inseefrlab)).  \
Make sure your cluster nodes are configured to pull from a mirror or prepull the corresponding images.  \
If needed, you can override the images Onyxia uses in the \`values.yaml\` and the images of your services in your catalogs \`values.yaml\` / \`values.schema.json\`&#x20;
