---
description: Instance Level Customization of the Service Catalog
icon: sliders-up
---

# values.schema.json overrides

This is the most common customization path.

It lets you change defaults and constraints without forking catalogs.

Use it to:

* set global policies (example: default RAM, max disk)
* restrict advanced options to specific roles (example: H100 only for users with a specific role assigned)

### Mental model

Some fields in a chart’s `values.schema.json` point to a schema file.

Onyxia ships a set of “well-known” schema files in the API:

{% embed url="https://github.com/InseeFrLab/onyxia-api/tree/main/onyxia-api/src/main/resources/schemas" %}

When a chart uses [`x-onyxia`](custom-catalogs/onyxia-extension.md)`.overwriteSchemaWith`, Onyxia resolves that schema path.

### How `overwriteSchemaWith` works

In the catalog charts (example: [InseeFrLab/helm-charts-interactive-services](https://github.com/inseefrlab/helm-charts-interactive-services)), look for `x-onyxia.overwriteSchemaWith` in `charts/*/values.schema.json`.

Example:

{% code title="charts/jupyter-python/values.schema.json (excerpt)" %}
```json
{
  "properties": {
    "service": {
      "properties": {
        "image": {
          "properties": {
            "custom": {
              "properties": {
                "enabled": {
                  "x-onyxia": {
                    "overwriteSchemaWith": "ide/customImage.json"
                  }
                }
              }
            }
          }
        }
      }
    }
  }
}
```
{% endcode %}

This means: “for this field, use the schema at `ide/customImage.json`”.

That schema file can come from:

* the default schemas embedded in Onyxia API
* an override you provide at the instance level (see below)

### Instance-wide overrides

Let's consider the [`ide/customImage.json`](https://github.com/InseeFrLab/onyxia-api/blob/main/onyxia-api/src/main/resources/schemas/ide/customImage.json) schema for exaple. By default this will be used:

{% code title="ide/customImage.json (default)" %}
```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "Use a custom image instead",
  "type": "boolean",
  "default": false
}
```
{% endcode %}

It enable users to provide a custom Docker image for a given service, let's say we want to remove this option. To do that you would configure your Onyxia instance like this: &#x20;

{% code title="apps/onyxia/values.yaml" %}
```yaml
onyxia:
  api:
    schemas:
      enabled: true
      files:
        - relativePath: ide/customImage.json
          content: |
            {
              "$schema": "http://json-schema.org/draft-07/schema#",
              "hidden": true,
              "const": false
            }
```
{% endcode %}

Result: the “custom image” toggle disappears from the launcher.

<details>

<summary>Other Example: change default resource sliders</summary>

This is a typical way to enforce sane defaults and limits for CPU/memory.

{% code title="apps/onyxia/values.yaml (excerpt)" %}
```yaml
onyxia:
  api:
    schemas:
      enabled: true
      files:
        - relativePath: ide/resources.json
          content: |
            {
              "$schema": "http://json-schema.org/draft-07/schema#",
              "title": "Resources",
              "description": "Your service will have at least the requested resources and never more than its limits.",
              "type": "object",
              "properties": {
                "requests": {
                  "description": "Guaranteed resources",
                  "type": "object",
                  "properties": {
                    "cpu": {
                      "title": "CPU",
                      "type": "string",
                      "default": "100m",
                      "render": "slider",
                      "sliderMin": 50,
                      "sliderMax": 10000,
                      "sliderStep": 50,
                      "sliderUnit": "m",
                      "sliderExtremity": "down",
                      "sliderExtremitySemantic": "guaranteed",
                      "sliderRangeId": "cpu"
                    },
                    "memory": {
                      "title": "Memory",
                      "type": "string",
                      "default": "2Gi",
                      "render": "slider",
                      "sliderMin": 1,
                      "sliderMax": 200,
                      "sliderStep": 1,
                      "sliderUnit": "Gi",
                      "sliderExtremity": "down",
                      "sliderExtremitySemantic": "guaranteed",
                      "sliderRangeId": "memory"
                    }
                  }
                },
                "limits": {
                  "description": "Max resources",
                  "type": "object",
                  "properties": {
                    "cpu": {
                      "title": "CPU",
                      "type": "string",
                      "default": "5000m",
                      "render": "slider",
                      "sliderMin": 50,
                      "sliderMax": 10000,
                      "sliderStep": 50,
                      "sliderUnit": "m",
                      "sliderExtremity": "up",
                      "sliderExtremitySemantic": "maximum",
                      "sliderRangeId": "cpu"
                    },
                    "memory": {
                      "title": "Memory",
                      "type": "string",
                      "default": "50Gi",
                      "render": "slider",
                      "sliderMin": 1,
                      "sliderMax": 200,
                      "sliderStep": 1,
                      "sliderUnit": "Gi",
                      "sliderExtremity": "up",
                      "sliderExtremitySemantic": "maximum",
                      "sliderRangeId": "memory"
                    }
                  }
                }
              }
            }
```
{% endcode %}

</details>

### Role-based overrides (different schema per user role)

Instance-wide overrides apply to everyone.

You can also apply schema overrides per role.

Onyxia reads roles from the decoded JWT access token.

By default, it uses the `roles` claim.

You can change that in your OIDC configuration.

See [OpenID Connect Configuration](../openid-connect-configuration.md).

#### Example: let `fullgpu` users choose H100

Here we override the built-in `nodeSelector-gpu.json` schema only for users with role `fullgpu`.

Other users still get the default schema.

{% code title="apps/onyxia/values.yaml" %}
```yaml
onyxia:
  api:
    schemas:
      enabled: true
      roles:
        - roleName: fullgpu
          files:
            - relativePath: nodeSelector-gpu.json
              content: |
                {
                  "$schema": "http://json-schema.org/draft-07/schema#",
                  "title": "Node Selector",
                  "type": "object",
                  "properties": {
                    "disktype": {
                      "description": "The type of disk",
                      "type": "string",
                      "enum": ["ssd", "hdd"],
                      "default": "ssd"
                    },
                    "gpu": {
                      "description": "The type of GPU",
                      "type": "string",
                      "enum": ["A2", "H100"],
                      "default": "A2"
                    }
                  },
                  "additionalProperties": false
                }
```
{% endcode %}

Result: `fullgpu` users can request GPU nodes and select `H100`.

### Next: user-specific defaults

So far you can override schemas:

* for everyone (instance-wide)
* for a subset of users (per role)

If you want per-user defaults (prefill from identity), use `x-onyxia.overwriteDefaultWith`.

Follow up with:

{% content-ref url="custom-catalogs/onyxia-extension.md" %}
[onyxia-extension.md](custom-catalogs/onyxia-extension.md)
{% endcontent-ref %}
