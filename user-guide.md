---
description: Using Onyxia (as a data scientist)
---

# 🕹 User guide

There are 3 main components accessible on the onyxia web interface :

* catalogs and services launched by the users (Kubernetes access)
* a file browser (S3 access)
* secret browser (Vault access)

## Start a service

Following is a documentation Onyxia when configured with the default service catalogs :&#x20;

* &#x20;[InseeFrLab/helm-charts-interactive-services](https://github.com/inseefrlab/helm-charts-interactive-services)

This collection of charts help users to launch many IDE with various binary stacks (python , R) with or without GPU support. Docker images are built [here](https://github.com/inseefrlab/images-datascience) and help us to give a homogeneous stack.

* &#x20;[InseeFrLab/helm-charts-databases](https://github.com/inseefrlab/helm-charts-databases)

This collection of charts help users to launch many databases system. Most of them are based on [bitnami/charts](https://guthub.com/bitnami/charts).

* &#x20;[InseeFrLab/helm-charts-automation](https://github.com/InseeFrLab/helm-charts-datascience)

This collection of charts help users to start automation tools for their datascience activity.&#x20;

{% hint style="info" %}
The Onyxia user experience may be very different from one catalog of service to another. &#x20;

The catalog defines what options are available though Onyxia. &#x20;
{% endhint %}

Users can edit various parameters. Onyxia do some assertion based on the charts values schema and the configuration on the instance. For example some identity token can be injected by default (because Onyxia connect users to many APIs).



![](<.gitbook/assets/image (11).png>)

After launching a service, notes are shown to the user. He can retrieve those notes on the README button. Charts administrator should explain how to connect to the services (url , account) and what happens on deletion.

![](<.gitbook/assets/image (2).png>)
