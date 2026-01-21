---
icon: microscope
---

# Catalog of services

By default Onyxia instances will features the following serices catalogs:

<table data-view="cards"><thead><tr><th></th><th data-type="content-ref"></th><th data-hidden data-card-cover data-type="image">Cover image</th></tr></thead><tbody><tr><td>Interactive Servicies (IDEs)</td><td><a href="https://github.com/inseefrlab/helm-charts-interactive-services">https://github.com/inseefrlab/helm-charts-interactive-services</a></td><td><a href="../../.gitbook/assets/Screenshot 2026-01-21 at 16.33.24.png">Screenshot 2026-01-21 at 16.33.24.png</a></td></tr><tr><td>Databases</td><td><a href="https://github.com/inseefrlab/helm-charts-databases">https://github.com/inseefrlab/helm-charts-databases</a></td><td><a href="../../.gitbook/assets/hero (1).webp">hero (1).webp</a></td></tr><tr><td>Automation</td><td><a href="https://github.com/InseeFrLab/helm-charts-automation/">https://github.com/InseeFrLab/helm-charts-automation/</a></td><td><a href="../../.gitbook/assets/mlflow-argo (2).png">mlflow-argo (2).png</a></td></tr><tr><td>Dataviz - Not enabled by default</td><td><a href="https://github.com/InseeFrLab/helm-charts-datavisualization">https://github.com/InseeFrLab/helm-charts-datavisualization</a></td><td><a href="../../.gitbook/assets/65dcd5fadc1df1b15af75c1e_Metabase vs Redash.png">65dcd5fadc1df1b15af75c1e_Metabase vs Redash.png</a></td></tr></tbody></table>

However as an Onyxia instance administrator there are many ways you can customize the experience of your users.

* You can change the default configuration of your services of your instance, for example adjust a default resource allocation policy.
* You can have different policies that apply on different group of users of your platforms. For example you can enforce that only a certain subset of user can allocate H100 to their services.
* You can even fork our base catalog, customize them, extend them or create your own service from scratch. Anything software that can be deployed on a kubernetes clusted can be turned into an onyxia service. See [Doom launched as an Onyxia service as an illustration](https://youtu.be/7SuXRfQqdGM?si=Y4_lXozJfh3ajyT7). &#x20;

## Base Principles

You can think of Onyxia as a graphical user interface for Helm.

### Baseline Facts - Not specific to Onyxia

A Helm repository is a collection of helm chart.

A helm chart is a recipe to deploy a software on kubernetes, you can think of it as an installer for K8s.

Each helm chart accept a certain number of configuration and knobs to parametrize the deployment. The default values are specified in the [values.yaml](https://github.com/InseeFrLab/helm-charts-interactive-services/blob/main/charts/jupyter-python/values.yaml) of the helm chart.

When deploying a specific helm chart on a kuberneted cluster you can overwirte any of those default values. &#x20;

The expected shape of of the values object used to launch the service can be specified by the chart author by providing a [values.schema.json](https://github.com/InseeFrLab/helm-charts-interactive-services/blob/main/charts/jupyter-python/values.schema.json). This essencially tells the user of the chart what are the available deployment option for a given chart and the expected format to specify them. &#x20;

### How Onyxia Levrages the Helm to deliver it's UX

The administrator of an Onyxia instance can configure Onyxia to specify which helm chart repositories should be used.

If no specific catalog are configured onyxia will load with [this configuration](https://github.com/InseeFrLab/onyxia-api/blob/main/onyxia-api/src/main/resources/catalogs.json) - Three catalog: Interactive Services - Databases - Automation.

Each helm chart repository will represent a tab in the "Service Catalog" page (Interactive Services, Databases, Automation...)

Each service card correspond to a specific helm chart (Jupyter-Python, RStudio, Jupyter-tensorflow).

<figure><img src="../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

Once on the launch page, Onyxia will read the values.schema.json of the selected chart and turn it into a graphical form so that user can see what option are available to configure a chart and adapt to their need without having to write a complex command. &#x20;

Aditionally, onyxia can inject relevent default in the configuration based on the identity of the user, like for example pre filling the user's S3 credential as shown here. &#x20;

<figure><img src="../../.gitbook/assets/onyxia-helm.png" alt=""><figcaption></figcaption></figure>

## Customization

Now that you get the biger picture let's see what are you options for customizing the catalog of your Onyxia Instance: &#x20;

<table data-card-size="large" data-view="cards"><thead><tr><th></th><th data-type="content-ref"></th></tr></thead><tbody><tr><td>Instance Level Configuration</td><td><a href="override-schema-for-a-specific-instance.md">override-schema-for-a-specific-instance.md</a></td></tr><tr><td>Helm Chart Repository Customization - Craft your own catalog of services</td><td><a href="custom-catalogs/">custom-catalogs</a></td></tr></tbody></table>
