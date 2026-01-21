---
icon: sliders-up
---

# values.schema.json overrides

This is the more common customization and the one recommended for most deployments. &#x20;

This approach provide a way, without going through the trouble of forking our helm chart catalog to changes specific defaults or allowed range for all the users of your platofrm OR for a specific subset of users.  &#x20;

This is what you will use for example to enforce that every instance should have 8Gb of ram allocated by default and no more than 100Gb. And make sure that only the useres of your platform that are in the "fullgpu" group can allocate H100 to their servicies. &#x20;

***

If you go in your chart catalogs that your Onyxia Instance is configured with. For example [inseeFrLab/helm-charts-datacience](https://github.com/inseefrlab/helm-charts-interactive-services). you can navigates in the different charts/\*/values.schema.json of your services and locate the overwriteSchemaWith: &#x20;

<pre class="language-json" data-title="charts/jupyter-python/values.schema.json"><code class="lang-json">{
  "properties": {
    "service": {
      "properties": {
        "image": {
          "properties": {
            "custom": {
              "properties": {
                "enabled": {
                  "x-onyxia": {
<strong>                    "overwriteSchemaWith": "ide/customImage.json"
</strong>                  }
                },
// ...
</code></pre>

An overwirteSchemaWith specify that the schema used for the property path (here service.image.custom.enabled) will be resolved from the specific Onyxia Instance that feature the chart as a service. &#x20;

You can find here the list of default well known shema that onyxia will use:

{% embed url="https://github.com/InseeFrLab/onyxia-api/tree/main/onyxia-api/src/main/resources/schemas" %}

Here it's:

{% code title="ide/customImage.json" %}
```json
{
    "$schema": "http://json-schema.org/draft-07/schema#",
    "title": "Use a custom image instead",
    "type": "boolean",
    "default": false
}    
```
{% endcode %}

The idea is that you can overwride this default shema for your instance. For example you can prevent user from being abble to use a custom image by configureing your onyxia deployment as such:

{% code title="apps/onyxia/values.yaml" %}
```yaml
onyxia:
  web:
    # ...
  api:
    # ...
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

This will have the effect of effectively removing the option of providing a custom image from the ui.

***

<details>

<summary>Another common usecase is to modify the resource allocation policy for the services lauched by the users of your instance.  </summary>

{% code title="onyxia-values.yaml" %}
```yaml
onyxia:
  web:
    # ...
  api:
    # ...
    schemas:
      enabled: true
      files:
        - relativePath: ide/resources.json
          content: |
            {
                "$schema": "http://json-schema.org/draft-07/schema#",
                "title": "Resources",
                "description": "Your service will have at least the requested resources and never more than its limits. No limit for a resource and you can consume everything left on the host machine.",
                "type": "object",
                "properties": {
                    "requests": {
                        "description": "Guaranteed resources",
                        "type": "object",
                        "properties": {
                            "cpu": {
                                "description": "The amount of cpu guaranteed",
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
                                "description": "The amount of memory guaranteed",
                                "title": "memory",
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
                        "description": "max resources",
                        "type": "object",
                        "properties": {
                            "cpu": {
                                "description": "The maximum amount of cpu",
                                "title": "CPU",
                                "type": "string",
                                "default": "5000m",
                                "render": "slider",
                                "sliderMin": 50,
                                "sliderMax": 10000,
                                "sliderStep": 50,
                                "sliderUnit": "m",
                                "sliderExtremity": "up",
                                "sliderExtremitySemantic": "Maximum",
                                "sliderRangeId": "cpu"
                            },
                            "memory": {
                                "description": "The maximum amount of memory",
                                "title": "Memory",
                                "type": "string",
                                "default": "50Gi",
                                "render": "slider",
                                "sliderMin": 1,
                                "sliderMax": 200,
                                "sliderStep": 1,
                                "sliderUnit": "Gi",
                                "sliderExtremity": "up",
                                "sliderExtremitySemantic": "Maximum",
                                "sliderRangeId": "memory"
                            }
                        }
                    }
                }
            }
```
{% endcode %}



</details>

***

So far we've only applied configuration that applies for all user of your platofrm.  \
Now let's see how to defines rules that differs depending of the role a given user has. &#x20;

Let's see how to apply different node selector policies for users of your platoforms that have the roles "fullgpu".  \
\
Before anything, note that onyxia will read the roles a given user has from the payload of the the decoded JWT access token. By default it will read the claims "roles" but this is configurable, see: [openid-connect-configuration.md](../openid-connect-configuration.md "mention"). &#x20;

&#x20;What we're going to do here is apply a specific override of the [nodeSelector.json](https://github.com/InseeFrLab/onyxia-api/blob/main/onyxia-api/src/main/resources/schemas/nodeSelector-gpu.json) schema. That will apply only for the user that have the "fullgpu" role. The other user will still get the default schema

{% code title="onyxia-values.yaml" %}
```yaml
onyxia:
  web:
    # ...
  api:
    # ...
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

This will have the effect of enabling a designated subset of user to have SSD allocated by default and an allocate an A2 nvidia GPU and optionally leave them the option to allocate a H100.

***

Now you know how to overrides the configuration for:

* All the user of your instance
* A group of user of your instance

The missing piece is to understand how to inject user specific defaults, follow up with:&#x20;

{% content-ref url="custom-catalogs/onyxia-extension.md" %}
[onyxia-extension.md](custom-catalogs/onyxia-extension.md)
{% endcontent-ref %}
