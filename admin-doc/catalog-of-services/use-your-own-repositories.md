---
description: Declare your own repository of charts
icon: house-flag
---

# Use your own repositories

If you do not specify catalogs in your `onyxia/values.yaml,` these are the ones that are used by default: [See file](https://github.com/InseeFrLab/onyxia-api/blob/main/onyxia-api/src/main/resources/catalogs.json).

To configure your onyxia instance to use your own custom helm repositories as onyxia catalogs you need to use the onyxia configuration `onyxia.api.catalogs`.  \
Let's say we're NASA and we want to have an "_Aerospace services"_ catalog on our onyxia instance. Our onyxia configuration would look a bit like this: &#x20;

{% code title="onyxia/values.yaml" %}
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
        status": "PROD",
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
