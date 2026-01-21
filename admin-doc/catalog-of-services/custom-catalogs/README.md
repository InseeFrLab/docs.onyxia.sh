---
description: Declare your own repository of charts
icon: house-flag
---

# Custom Catalogs

Use custom catalogs when you want to:

* fork the official catalogs and maintain your own variants
* publish internal charts (private org tooling)
* expose non-official charts as first-class Onyxia services

{% hint style="info" %}
If you don’t configure `onyxia.api.catalogs`, Onyxia loads the defaults from [`catalogs.json`](https://github.com/InseeFrLab/onyxia-api/blob/main/onyxia-api/src/main/resources/catalogs.json).
{% endhint %}

{% hint style="info" %}
If you only need to change defaults/constraints, avoid forking catalogs. Use [values.schema.json overrides](../override-schema-for-a-specific-instance.md).
{% endhint %}

### Configure catalogs

Catalogs are configured in `apps/onyxia/values.yaml` under `onyxia.api.catalogs`.

Example: you’re NASA and you want an “Aerospace services” tab.

{% code title="apps/onyxia/values.yaml" %}
```yaml
onyxia:
  web:
    # ...
  api:
    # ...
    catalogs: [
      {
        type: "helm",
        id: "aerospace",
        # The url of the Helm chart repository
        location: "https://myorg.github.io/helm-charts-aerospace/",
        # Display under the search bar as selection tab:
        # https://github.com/InseeFrLab/onyxia/assets/6702424/a7247c7d-b0be-48db-893b-20c9352fdb94
        name: { 
          en: "Aerospace services",
          fr: "Services aérospatiaux"
          # ... other languages your instance supports
        },
        # Optional. Defines the chart that should appear first
        highlightedCharts: ["jupyter-artemis", "rstudio-dragonfly"],
        # Optional. Defines the chart that should be excluded
        excludedCharts: ["a-vendor-locking-chart"],
        # Optional, If defined, displayed in the header of the catalog page:
        # https://github.com/InseeFrLab/onyxia/assets/6702424/57e32f44-b889-41b2-b0c7-727c35b07650
        # Is rendered as Markdown
        description: { 
          en: "A catalog of services for aerospace engineers",
          fr: "Un catalogue de services pour les ingénieurs aérospatiaux"
          # ...
        },
        # Can be "PROD" or "TEST". If test the catalogs will be accessible if you type the url in the search bar
        # but you won't have a tab to select it.
        status: "PROD",
        # Optional. If true the certificate verification for `${location}/index.yaml` will be skipped.
        skipTlsVerify: false,
        # Optional. certificate authority file to use for the TLS verification
        caFile: "/path/to/ca.crt",
        # Optional: Enables you to a specific group of users.
        # You can match any claim in the JWT token.  
        # If the claim's value is an array, it match if one of the value is the one you specified.
        # The match property can also be a regex.
        restrictions: [
          {
            userAttribute: {
              key: "groups",
              matches: "nasa-engineers"
            }
          }
        ]
      },
       # { ... } another catalog
    ]
```
{% endcode %}

### Next: fork or build a catalog repo

Most setups start by forking an official catalog and editing `charts/*/values.schema.json` and chart defaults.

Good starting point: [InseeFrLab/helm-charts-interactive-services](https://github.com/inseefrlab/helm-charts-interactive-services).

To go further, you’ll want the Onyxia JSON Schema extensions:

{% content-ref url="onyxia-extension.md" %}
[onyxia-extension.md](onyxia-extension.md)
{% endcontent-ref %}
