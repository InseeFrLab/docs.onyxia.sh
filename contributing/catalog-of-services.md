---
description: Contributing to the helm charts
---

# 🔬 Catalog of services



Every Onyxia instance may or may not have it's own catalog. &#x20;

If you take for example [this instance](https://datalab.sspcloud.fr/catalog) it offers two catalogs, [InseeFrLab datascience](https://github.com/InseeFrLab/helm-charts-datascience) and [InseeFrLab R Shiny apps](https://github.com/InseeFrLab/helm-charts-shiny-apps). &#x20;

You can always find the source of the catalog by clicking on the "contribute to the... " link.

![https://datalab.sspcloud.fr/catalog](<../.gitbook/assets/image (5).png>)

If you take [this other instance](https://sill-demo.etalab.gouv.fr), it has only one catalog, [helm-charts-sill](https://github.com/etalab/helm-charts-sill).

![https://sill-demo.etalab.gouv.fr/catalog](<../.gitbook/assets/image (9).png>)

In order to contribute you have to be familiar with [Helm](https://helm.sh/) and to be familiar with Helm you need to be familiar with [Kubernetes objects](https://kubernetes.io/docs/concepts/overview/working-with-objects/kubernetes-objects/). &#x20;

In Onyxia, we use the `values.shema.json` to know what options should be displayed to the user at [the service configuration step](https://user-images.githubusercontent.com/6702424/177571819-f2e1b4ef-ecd1-479b-a5a1-658d87d7c7c0.png) and what default value Onyxia should inject. &#x20;

![https://helm.sh/docs/topics/charts/#the-chart-file-structure](<../.gitbook/assets/image (6).png>)

How it works in practice is that, in a Helm chart made for Onyxia the values.sheama.json isn't actually a JSON file per say but rather a [.mustache](https://mustache.github.io/) template that renders into a JSON file. &#x20;

{% embed url="https://cloud.githubusercontent.com/assets/288977/8779228/a3cf700e-2f02-11e5-869a-300312fb7a00.gif" %}
